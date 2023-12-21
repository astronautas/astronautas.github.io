---
layout: post
title: Let's build a Bloom filter. Part 1
---
I have always been intrigued by Bloom filters. They are very similar to hash sets but somehow much more space-efficient. Sometimes, they can yield false positives, creating a peculiar tradeoff.

Let's explore what makes Bloom filters tick, starting with a high-level comparison to a hash set. As a reminder, a hash set is a hash map whose value is empty, allowing it only to check whether an item has been inserted within it or not. Similarly, Bloom filters are used solely for membership queries.

![img](/assets/image-2.png)

<p>
<center><i>A Hash set</i></center>
</p>


![img](/assets/image-3.png)

<center><i>A Bloom filter</i></center>

<p></p>

**So, how much space we can actually save?**

* **Hash set:** Requires `4 bytes * N` for 32-bit keys, such as integers.

* **Bloom filter:** Utilizes `(m (filter size in bits) / 8 bits) * N` bytes, as it doesn't store the keys.

In practice, `m` is often set to be <a href="https://hur.st/bloomfilter/?n=5000&p=0.0001&m=&k=6">20 times the element count</a>. For a `false positive rate (FP_rate)` of 0.0001, we can save anywhere from 37% (using 32-bit ints) to 75% (with 10-character string keys). Excellent! The catch here is, once again (crucial to grasp) - <u>Bloom filters do not store keys</u>, so, unlike for hash sets, collisions are acceptable, and there's no need to look up within the buckets.

**How do we control FP rate?**

To control the false positive rate, one can increase the value of 'm.' In practice as well, Bloom filters employ multiple hash functions when hashing keys to ensure better uniformity.

Let's gain a better understanding of Bloom filters by implementing one, naturally, in Rust 🦀:

```rust
use sha2::{Sha256, Digest};

struct BloomFilter {
    state: u8,
}

impl BloomFilter {
    fn get_bucket(val: &str) -> u8 {
        let mut hasher = Sha256::new();
        hasher.update(val);
        let hash = hasher.finalize();

        let mut hash_int: u128 = 0;

        for i in 0..16 {
            hash_int = (hash_int << 8) + hash[i] as u128;
        }

        let bucket = hash_int % 8;

        return bucket as u8;
    }

    fn new() -> Self {
        BloomFilter {
            state: 0
        }
    }

    fn insert(&mut self, val: &str) {
        let bucket = BloomFilter::get_bucket(val);

        // getting the right bit
        // duplication
        let mut mask: u8 = 1;

        for i in 0..bucket {
            mask = mask << 1;
        }

        self.state = self.state | mask;
    }

    fn get(&self, val: &str) -> bool {

        let bucket = BloomFilter::get_bucket(val);

        // getting the right bit
        let mut mask: u8 = 1;

        for i in 0..bucket {
            mask = mask << 1;
        }

        // e.g. only if all same bits are set in the mask (checking), then it means value
        // has been set
        return (mask & self.state) == mask
    }
}

fn main() {
    let mut bloom_filter = BloomFilter::new();

    bloom_filter.insert("bla_bla");

    println!("{}", bloom_filter.get("bla_bla"));
    // so if it says false, then it means 100% it's not in the set. true means it might be there, but not guaranteed.
    println!("{}", bloom_filter.get("bla_bla_6"));
}
```

```rust
struct BloomFilter {
    state: u8,
}
```

Our filter relies on bit-backed buckets, a departure from hash sets using an array for buckets (since again we do not store keys). We use bits to indicate whether a bucket holds an element. The 8-bit limit in the code above simplifies the initial implementation, addressing challenges in directly manipulating bits on most architectures. Constructing a bitset from smaller memory units is a significant hassle.

Also, if you zoom in on the `insert` and `get` methods, you will notice that to get/set individual bits, we need to perform bit shift operations. This is because, once again, the smallest memory unit is a byte, and we cannot address individual bits directly.

```rust
fn main() {
    let mut bloom_filter = BloomFilter::new();

    bloom_filter.insert("bla_bla");

    println!("{}", bloom_filter.get("bla_bla"));
    // so if it says false, then it means 100% it's not in the set. true means it might be there, but not guaranteed.
    println!("{}", bloom_filter.get("bla_bla_6"));
}
```

If you run the code above, you will likely get `true` and `false`. If you change values, you might notice that quite often the filter says a value exists, whereas we never actually added it. To address this, in the next chapter, we will attempt to reduce the false-positive rate by manipulating the Bloom filter's size (`m`) and introducing additional hash functions to increase the uniformity of the value set (`k`).

Feel free to experiment with the implementation in the workspace below. Full source is available as a <a href="https://github.com/astronautas/bloom-filter-rs">public repository</a>.

<iframe
    width="100%"
    height="600"
    src="https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&code=use+sha2%3A%3A%7BSha256%2C+Digest%7D%3B%0A%0Astruct+BloomFilter+%7B%0A++++state%3A+u8%2C%0A%7D%0A%0Aimpl+BloomFilter+%7B%0A++++fn+get_bucket%28val%3A+%26str%29+-%3E+u8+%7B%0A++++++++let+mut+hasher+%3D+Sha256%3A%3Anew%28%29%3B%0A++++++++hasher.update%28val%29%3B%0A++++++++let+hash+%3D+hasher.finalize%28%29%3B%0A%0A++++++++let+mut+hash_int%3A+u128+%3D+0%3B%0A%0A++++++++for+i+in+0..16+%7B%0A++++++++++++hash_int+%3D+%28hash_int+%3C%3C+8%29+%2B+hash%5Bi%5D+as+u128%3B%0A++++++++%7D%0A%0A++++++++let+bucket+%3D+hash_int+%25+8%3B%0A%0A++++++++return+bucket+as+u8%3B%0A++++%7D%0A%0A++++fn+new%28%29+-%3E+Self+%7B%0A++++++++BloomFilter+%7B%0A++++++++++++state%3A+0%0A++++++++%7D%0A++++%7D%0A%0A++++fn+insert%28%26mut+self%2C+val%3A+%26str%29+%7B%0A++++++++let+bucket+%3D+BloomFilter%3A%3Aget_bucket%28val%29%3B%0A%0A++++++++%2F%2F+getting+the+right+bit%0A++++++++%2F%2F+duplication%0A++++++++let+mut+mask%3A+u8+%3D+1%3B%0A%0A++++++++for+i+in+0..bucket+%7B%0A++++++++++++mask+%3D+mask+%3C%3C+1%3B%0A++++++++%7D%0A%0A++++++++self.state+%3D+self.state+%7C+mask%3B%0A++++%7D%0A%0A++++fn+get%28%26self%2C+val%3A+%26str%29+-%3E+bool+%7B%0A%0A++++++++let+bucket+%3D+BloomFilter%3A%3Aget_bucket%28val%29%3B%0A%0A++++++++%2F%2F+getting+the+right+bit%0A++++++++let+mut+mask%3A+u8+%3D+1%3B%0A%0A++++++++for+i+in+0..bucket+%7B%0A++++++++++++mask+%3D+mask+%3C%3C+1%3B%0A++++++++%7D%0A%0A++++++++%2F%2F+e.g.+only+if+all+same+bits+are+set+in+the+mask+%28checking%29%2C+then+it+means+value%0A++++++++%2F%2F+has+been+set%0A++++++++return+%28mask+%26+self.state%29+%3D%3D+mask%0A++++%7D%0A%7D%0A%0Afn+main%28%29+%7B%0A++++let+mut+bloom_filter+%3D+BloomFilter%3A%3Anew%28%29%3B%0A%0A++++bloom_filter.insert%28%22bla_bla%22%29%3B%0A%0A++++println%21%28%22%7B%7D%22%2C+bloom_filter.get%28%22bla_bla%22%29%29%3B%0A++++%2F%2F+so+if+it+says+false%2C+then+it+means+100%25+it%27s+not+in+the+set.+true+means+it+might+be+there%2C+but+not+guaranteed.%0A++++println%21%28%22%7B%7D%22%2C+bloom_filter.get%28%22bla_bla_5%22%29%29%3B%0A%7D"
    frameborder="0"
    allowfullscreen
></iframe>
