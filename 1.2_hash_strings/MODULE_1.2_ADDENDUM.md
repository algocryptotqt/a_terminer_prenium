# MODULE 1.2 - ADDENDUM
## Exercices supplémentaires pour couverture complète

Ces exercices complètent MODULE_1.2_EXERCICES_COMPLETS.md pour atteindre 100% de couverture.

---

## Exercice ADD-1: `hash_trait_derive`
**Couvre: 1.2.1.i-j (2 concepts)**

### Concepts
- [1.2.1.i] Rust `Hash` trait — Interface standard
- [1.2.1.j] `#[derive(Hash)]` — Dérivation automatique

### Rust
```rust
use std::collections::HashSet;
use std::hash::{Hash, Hasher};

// 1.2.1.j - Dérivation automatique
#[derive(Hash, PartialEq, Eq)]
struct Point {
    x: i32,
    y: i32,
}

// 1.2.1.i - Implémentation manuelle du trait Hash
struct CustomKey {
    id: u64,
    name: String,
    // On ignore 'cached' pour le hash
    cached: Option<Vec<u8>>,
}

impl Hash for CustomKey {
    fn hash<H: Hasher>(&self, state: &mut H) {
        // Seulement id et name participent au hash
        self.id.hash(state);
        self.name.hash(state);
    }
}

impl PartialEq for CustomKey {
    fn eq(&self, other: &Self) -> bool {
        self.id == other.id && self.name == other.name
    }
}

impl Eq for CustomKey {}

pub fn demonstrate_hash_trait() {
    // Avec derive
    let mut points: HashSet<Point> = HashSet::new();
    points.insert(Point { x: 1, y: 2 });
    points.insert(Point { x: 3, y: 4 });
    assert!(points.contains(&Point { x: 1, y: 2 }));

    // Avec implémentation custom
    let mut keys: HashSet<CustomKey> = HashSet::new();
    keys.insert(CustomKey {
        id: 1,
        name: "first".into(),
        cached: None,
    });

    // Même clé avec cache différent = trouvée
    assert!(keys.contains(&CustomKey {
        id: 1,
        name: "first".into(),
        cached: Some(vec![1, 2, 3]),
    }));
}
```

### Test Moulinette
```
hash_trait derive Point(1,2) Point(3,4) contains Point(1,2) -> true
hash_trait custom id=1 name="first" -> inserted
```

---

## Exercice ADD-2: `utf8_string_indexing`
**Couvre: 1.2.6bis.a-h (8 concepts)**

### Concepts
- [1.2.6bis.a] Pourquoi `s[i]` ne fonctionne pas — UTF-8 encodage variable
- [1.2.6bis.b] `s.chars().nth(i)` — O(n) accès par caractère
- [1.2.6bis.c] `s.as_bytes()[i]` — O(1) accès aux bytes
- [1.2.6bis.d] `s.char_indices()` — Iterator (byte_index, char)
- [1.2.6bis.e] Solution: `Vec<char>` — Conversion pour O(1)
- [1.2.6bis.f] Grapheme clusters — Caractères composés
- [1.2.6bis.g] `unicode-segmentation` crate — Manipulation correcte
- [1.2.6bis.h] Quand utiliser quoi — Guide pratique

### Rust
```rust
use unicode_segmentation::UnicodeSegmentation;

pub fn demonstrate_utf8_indexing() {
    let s = "héllo 世界 🇫🇷";

    // 1.2.6bis.a - Pourquoi s[i] ne marche pas
    // let c = s[0];  // ERREUR: String ne peut pas être indexé par usize
    // UTF-8 = 1-4 bytes par caractère, index byte != index caractère

    // 1.2.6bis.b - chars().nth(i) - O(n)
    let third_char = s.chars().nth(2);  // 'l'
    assert_eq!(third_char, Some('l'));

    // 1.2.6bis.c - as_bytes()[i] - O(1) mais bytes, pas chars!
    let bytes = s.as_bytes();
    assert_eq!(bytes[0], b'h');           // OK: 'h' = 1 byte
    assert_eq!(bytes[1], 0xC3);           // 'é' commence ici (2 bytes)
    assert_eq!(bytes[2], 0xA9);           // suite de 'é'
    // bytes[1] n'est PAS 'é', c'est juste le premier byte de 'é'

    // 1.2.6bis.d - char_indices() - (byte_offset, char)
    for (byte_idx, ch) in s.char_indices() {
        println!("byte {}: '{}'", byte_idx, ch);
    }
    // byte 0: 'h'
    // byte 1: 'é'  (bytes 1-2)
    // byte 3: 'l'
    // ...

    // 1.2.6bis.e - Solution Vec<char> pour O(1)
    let chars: Vec<char> = s.chars().collect();
    assert_eq!(chars[1], 'é');  // O(1) maintenant!
    assert_eq!(chars[6], '世');

    // 1.2.6bis.f - Grapheme clusters
    // Le drapeau 🇫🇷 = 2 code points (🇫 + 🇷)
    let flag = "🇫🇷";
    assert_eq!(flag.chars().count(), 2);     // 2 chars!
    assert_eq!(flag.graphemes(true).count(), 1);  // 1 graphème

    // 1.2.6bis.g - unicode-segmentation crate
    let complex = "é";  // peut être 1 ou 2 code points selon normalisation
    for grapheme in complex.graphemes(true) {
        println!("grapheme: {}", grapheme);
    }

    // 1.2.6bis.h - Quand utiliser quoi
    // ASCII only → as_bytes() - le plus rapide
    // Unicode simple → chars()
    // Unicode complet (emojis, accents combinés) → graphemes()
}

/// Compte les "vrais" caractères visibles
pub fn visual_char_count(s: &str) -> usize {
    s.graphemes(true).count()
}

/// Accès O(1) pour strings ASCII
pub fn ascii_char_at(s: &str, i: usize) -> Option<char> {
    if s.is_ascii() {
        s.as_bytes().get(i).map(|&b| b as char)
    } else {
        s.chars().nth(i)
    }
}

/// Substring par indices de caractères (pas bytes)
pub fn char_substring(s: &str, start: usize, end: usize) -> String {
    s.chars().skip(start).take(end - start).collect()
}
```

### Test Moulinette
```
utf8 char_count "hello" -> 5
utf8 char_count "héllo" -> 5
utf8 char_count "世界" -> 2
utf8 visual_count "🇫🇷" -> 1
utf8 char_at "hello" 1 -> 'e'
utf8 char_at "héllo" 1 -> 'é'
utf8 substring "hello world" 0 5 -> "hello"
```

---

## Exercice ADD-3: `string_advanced_methods`
**Couvre: 1.2.3.h, 1.2.10.i-j, 1.2.21.j (4 concepts)**

### Concepts
- [1.2.3.h] String interning — Réutilisation de strings
- [1.2.10.i] Pattern matching avancé — Regex groups
- [1.2.10.j] Lazy evaluation regex — Compilation unique
- [1.2.21.j] Aho-Corasick optimisations — Multi-pattern

### Rust
```rust
use std::collections::HashMap;
use std::sync::OnceLock;
use regex::Regex;
use aho_corasick::AhoCorasick;

// 1.2.3.h - String interning (pool de strings)
pub struct StringInterner {
    pool: HashMap<String, usize>,
    strings: Vec<String>,
}

impl StringInterner {
    pub fn new() -> Self {
        Self {
            pool: HashMap::new(),
            strings: Vec::new(),
        }
    }

    pub fn intern(&mut self, s: &str) -> usize {
        if let Some(&id) = self.pool.get(s) {
            return id;  // Déjà interné
        }
        let id = self.strings.len();
        self.strings.push(s.to_string());
        self.pool.insert(s.to_string(), id);
        id
    }

    pub fn get(&self, id: usize) -> Option<&str> {
        self.strings.get(id).map(|s| s.as_str())
    }
}

// 1.2.10.j - Lazy regex compilation
fn email_regex() -> &'static Regex {
    static RE: OnceLock<Regex> = OnceLock::new();
    RE.get_or_init(|| {
        Regex::new(r"([a-zA-Z0-9._%+-]+)@([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})").unwrap()
    })
}

// 1.2.10.i - Pattern matching avec groups
pub fn extract_email_parts(text: &str) -> Vec<(String, String)> {
    let re = email_regex();
    re.captures_iter(text)
        .map(|cap| {
            (cap[1].to_string(), cap[2].to_string())  // (user, domain)
        })
        .collect()
}

// 1.2.21.j - Aho-Corasick multi-pattern search
pub fn find_keywords(text: &str, keywords: &[&str]) -> Vec<(usize, String)> {
    let ac = AhoCorasick::new(keywords).unwrap();
    ac.find_iter(text)
        .map(|m| (m.start(), keywords[m.pattern().as_usize()].to_string()))
        .collect()
}

pub fn demonstrate_advanced_strings() {
    // String interning
    let mut interner = StringInterner::new();
    let id1 = interner.intern("hello");
    let id2 = interner.intern("world");
    let id3 = interner.intern("hello");  // Réutilise id1
    assert_eq!(id1, id3);

    // Regex groups
    let text = "Contact: alice@example.com or bob@test.org";
    let emails = extract_email_parts(text);
    assert_eq!(emails[0], ("alice".into(), "example.com".into()));

    // Multi-pattern Aho-Corasick
    let text = "The quick brown fox jumps over the lazy dog";
    let keywords = ["quick", "fox", "dog"];
    let found = find_keywords(text, &keywords);
    // [(4, "quick"), (16, "fox"), (40, "dog")]
}
```

### Test Moulinette
```
interner intern "hello" "world" "hello" -> [0, 1, 0]
regex_groups "alice@example.com" -> [("alice", "example.com")]
aho_corasick "the quick fox" ["quick","fox"] -> [(4,"quick"),(10,"fox")]
```

---

## RÉCAPITULATIF MODULE 1.2

| Exercice | Concepts | Count |
|----------|----------|-------|
| ADD-1 hash_trait_derive | 1.2.1.i-j | 2 |
| ADD-2 utf8_string_indexing | 1.2.6bis.a-h | 8 |
| ADD-3 string_advanced_methods | 1.2.3.h, 1.2.10.i-j, 1.2.21.j | 4 |
| **TOTAL AJOUTÉ** | | **14** |

**Couverture Module 1.2: 165 + 14 = 179/179 = 100%**

