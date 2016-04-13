# `strbuf` library

(Originally used in [git](http://github.com/git/git) and as described in `git`)

strbuf's are meant to be used with all the usual C string and memory
APIs. Given that the length of the buffer is known, it's often better to
use the mem* functions than a str* one (memchr vs. strchr e.g.).
Though, one has to be careful about the fact that str* functions often
stop on NULs and that strbufs may have embedded NULs.

A strbuf is NUL terminated for convenience, but no function in the
strbuf API actually relies on the string being free of NULs.

strbufs have some invariants that are very important to keep in mind:

  - The `buf` member is never NULL, so it can be used in any usual C
    string operations safely. strbuf's _have_ to be initialized either by
    `strbuf_init()` or by `= STRBUF_INIT` before the invariants, though.

    Do *not* assume anything on what `buf` really is (e.g. if it is
    allocated memory or not), use `strbuf_detach()` to unwrap a memory
    buffer from its strbuf shell in a safe way. That is the sole supported
    way. This will give you a malloced buffer that you can later `free()`.

    However, it is totally safe to modify anything in the string pointed by
    the `buf` member, between the indices `0` and `len-1` (inclusive).

  - The `buf` member is a byte array that has at least `len + 1` bytes
    allocated. The extra byte is used to store a `'\0'`, allowing the
    `buf` member to be a valid C-string. Every strbuf function ensure this
    invariant is preserved.

**NOTE**: It is OK to "play" with the buffer directly if you work it this
    way:

    ```C
        strbuf_grow(sb, SOME_SIZE); <1>
        strbuf_setlen(sb, sb->len + SOME_OTHER_SIZE);
    ```

<1> Here, the memory array starting at `sb->buf`, and of length
    `strbuf_avail(sb)` is all yours, and you can be sure that
    `strbuf_avail(sb)` is at least `SOME_SIZE`.

**NOTE**: `SOME_OTHER_SIZE` must be smaller or equal to `strbuf_avail(sb)`.

Doing so is safe, though if it has to be done in many places, adding the
missing API to the strbuf module is the way to go.

**WARNING**: Do _not_ assume that the area that is yours is of size
`alloc - 1` even if it's true in the current implementation. Alloc is
somehow a _private_ member that should not be messed with. Use `strbuf_avail()` instead.

