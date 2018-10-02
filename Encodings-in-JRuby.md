# Character length
* For now this a small wrap-up of usage and semantics of MRI's character length function plenitude
```c

// used by Onigmo internals (parsing, compiling, execution, some encodings)
#define enclen(enc,p,e) ((enc->max_enc_len == enc->min_enc_len) ? enc->min_enc_len : ONIGENC_MBC_ENC_LEN(enc,p,e))

// encoding.c, regenc.c
#define ONIGENC_MBC_ENC_LEN(enc,p,e)           onigenc_mbclen_approximate(p,e,enc)

// regenc.c only
extern int onigenc_mbclen_approximate(const OnigUChar* p,const OnigUChar* e, OnigEncoding enc) {
  int ret = ONIGENC_PRECISE_MBC_ENC_LEN(enc, p, e);
  if (ONIGENC_MBCLEN_CHARFOUND_P(ret))
    return ONIGENC_MBCLEN_CHARFOUND_LEN(ret);
  else if (ONIGENC_MBCLEN_NEEDMORE_P(ret))
    return (int )(e - p) + ONIGENC_MBCLEN_NEEDMORE_LEN(ret);
  return 1;
}

// used by unicode.c, encoding.c, regenc.c
#define ONIGENC_PRECISE_MBC_ENC_LEN(enc,p,e)   (enc)->precise_mbc_enc_len(p,e,enc)

// used in re.c
#define mbclen(p,e,enc)  rb_enc_mbclen((p),(e),(enc))

// used by core and exts
int rb_enc_precise_mbclen(const char *p, const char *e, rb_encoding *enc); /* -> chlen, invalid or needmore */


// used by core and exts
int rb_enc_mbclen(const char *p, const char *e, rb_encoding *enc) {
    int n = ONIGENC_PRECISE_MBC_ENC_LEN(enc, (UChar*)p, (UChar*)e);
    if (MBCLEN_CHARFOUND_P(n) && MBCLEN_CHARFOUND_LEN(n) <= e-p)
        return MBCLEN_CHARFOUND_LEN(n);
    else {
        int min = rb_enc_mbminlen(enc);
        return min <= e-p ? min : (int)(e-p);
    }
}

// used by unicode.c, encoding.c, regenc.c
int rb_enc_precise_mbclen(const char *p, const char *e, rb_encoding *enc) {
    int n;
    if (e <= p)
        return ONIGENC_CONSTRUCT_MBCLEN_NEEDMORE(1);
    n = ONIGENC_PRECISE_MBC_ENC_LEN(enc, (UChar*)p, (UChar*)e);
    if (e-p < n)
        return ONIGENC_CONSTRUCT_MBCLEN_NEEDMORE(n-(int)(e-p));
    return n;
}

// used by parser
#define parser_mbclen()  mbclen((lex_p-1),lex_pend,current_enc)

// used by parser
static int parser_precise_mbclen(struct parser_params *parser, const char *p) {
    int len = rb_enc_precise_mbclen(p, lex_pend, current_enc);
    if (!MBCLEN_CHARFOUND_P(len)) {
  compile_error(PARSER_ARG "invalid multibyte char (%s)", parser_encoding_name());
  return -1;
    }
    return len;
}

// unused
#define onig_enc_len(enc,p,e)          ONIGENC_MBC_ENC_LEN(enc, p, e)

```