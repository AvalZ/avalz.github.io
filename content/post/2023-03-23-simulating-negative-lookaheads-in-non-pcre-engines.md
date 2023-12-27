---
cover:
  image: /assets/posts/negative-lookahead/confused-travolta.jpg
  alt: "Confused Travolta"
categories:
- Tricks
date: "2023-03-23T00:00:00Z"
excerpt: Negative lookaheads (and lookaround constructs in general) are an awesome
  feature in PCRE. But what if you are using a non-PCRE engine?
tags:
- github
- go
- regex
- rust
title: Simulating Negative Lookaheads in non-PCRE Engines
---

Negative lookaheads (and lookaround constructs in general) are an awesome feature in PCRE.

For example, to match all instances of `abc`, but only those that are NOT followed by `def`, you can use the following regex:

```
abc(?!def)
```

However, some modern regex engines do not support lookaround constructs due to [performance concerns](https://github.com/google/re2/issues/255). This is the case for the default regex engines in Go and Rust (for example, GitHub uses Rust, and so do its search features).

# A Quick Fix

Fortunately, there is a sneaky way to simulate this behavior, although it does involve using a lot of characters.

When we use `(?!def)`, we are essentially saying "I don't want to see `def` after this, but I would be okay with seeing something like `deg` or `daf`".

To put it more formally, we can break it down into a series of statements:

1. I don't want a `d` after this word.
2. However, I would be okay with a `d` if it's not followed by an `e`.
3. I would also be okay with `de`, but only if it's not followed by an `f`.

These statements are easier to write in regular engines because we can use the negative character range (`[^ ]`) construct. For example, `[^a]` would mean "everything but `a`".

So the equivalent regular expression to `(?!def)` would be:

`[^d]|d[^e]|de[^f]`

For longer expressions, you can use a quick Python script like this:

```
look = "def"
"|".join([f"{look[:i]}[^{look[i]}]" for i in range(len(look))])
	# Output =>    [^d]|d[^e]|de[^f]

```

# The Pitfalls of Negative Alternations

Sometimes, one negative lookahead is not enough to exclude a match. If we want to exclude multiple words, we can use an alternation inside the negative lookahead construct. For example, to exclude matches for `def`, `ghi`, and `jkl`, we can use the following negative lookahead.

```
(?!(def|ghi|jkl))
```

In this case, one might be tempted to simply replicate the previous approach and alternate everything.

```
[^d]|d[^e]|de[^f]|[^g]|g[^h]|gh[^i]|[^j]|j[^k]|jk[^l]
```

However, by doing this, we inadvertently create a trap that allows any character.

In particular, if we group the initial blocks of all alternations we get this: `[^d]|[^g]|[^j]`. The problem is that this specific alternation is the equivalent of “any character”, because we are saying “anything but `d`, OR anything but `g`, OR anything but `j`". For example, if we had `def` as input, this would fail on the first branch of the alternation (the first character is indeed `d`), but it would pass on the second branch (since it accepts anything but `g`). 

To solve this, we can simply group these negative matching groups into one negative range.

```
[^dgj]|d[^e]|de[^f]|g[^h]|gh[^i]|j[^k]|jk[^l]
```

Of course, this should be repeated in case some words have similar initial patterns (e.g., `def` and `deg`)

# Limitations

This technique only works for "immediate" negative lookahead. It is effective if we want to exclude a string that immediately follows a word. However, if we want to exclude a word that occurs "somewhere" after our target word, this method may not work as well.

Also, this could cause performance issues if the regex becomes too big (this needs further testing).
