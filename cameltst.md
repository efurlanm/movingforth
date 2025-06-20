```
; Listing 1.
; ===============================================
; CamelForth for the Zilog Z80
; Primitive testing code
;
; This is the "minimal" test of the CamelForth
; kernel.  It verifies the threading and nesting
; mechanisms, the stacks, and the primitives
;   DUP EMIT EXIT lit branch ONEPLUS.
; It is particularly useful because it does not
; use the DO..LOOP, multiply, or divide words,
; and because it can be used on embedded CPUs.
; The numeric display word .A is also useful
; for testing the rest of the Core wordset.
;
; The required macros and CPU initialization
; are in file CAMEL80.AZM.
; ===============================================

;Z ><   u1 -- u2    swap the bytes of TOS
    head SWAB,2,><,docode
        ld a,b
        ld b,c
        ld c,a
        next

;Z LO   c1 -- c2    return low nybble of TOS
    head LO,2,LO,docode
        ld a,c
        and 0fh
        ld c,a
        ld b,0
        next

;Z HI   c1 -- c2    return high nybble of TOS
    head HI,2,HI,docode
        ld a,c
        and 0f0h
        rrca
        rrca
        rrca
        rrca
        ld c,a
        ld b,0
        next

;Z >HEX  c1 -- c2    convert nybble to hex char
    head TOHEX,4,>HEX,docode
        ld a,c
        sub 0ah
        jr c,numeric
        add a,7
numeric: add a,3ah
        ld c,a
        next

;Z .HH   c --       print byte as 2 hex digits
;   DUP HI >HEX EMIT LO >HEX EMIT ;
    head DOTHH,3,.HH,docolon
        DW DUP,HI,TOHEX,EMIT,LO,TOHEX,EMIT,EXIT

;Z .B    a -- a+1   fetch & print byte, advancing
;   DUP C@ .HH 20 EMIT 1+ ;
    head DOTB,2,.B,docolon
    DW DUP,CFETCH,DOTHH,lit,20h,EMIT,ONEPLUS,EXIT

;Z .A   u --       print unsigned as 4 hex digits
;   DUP >< .HH .HH 20 EMIT ;
    head DOTA,2,.A,docolon
        DW DUP,SWAB,DOTHH,DOTHH,lit,20h,EMIT,EXIT

;X DUMP   addr u --      dump u locations at addr
;   0 DO
;      I 15 AND 0= IF CR DUP .A THEN
;      .B
;   LOOP DROP ;
    head DUMP,4,DUMP,docolon
        DW LIT,0,XDO
DUMP2:  DW II,LIT,15,AND,ZEROEQUAL,qbranch,DUMP1
        DW CR,DUP,DOTA
DUMP1:  DW DOTB,XLOOP,DUMP2,DROP,EXIT

;Z ZQUIT   --    endless dump for testing
;   0 BEGIN  0D EMIT 0A EMIT  DUP .A
;       .B .B .B .B .B .B .B .B
;       .B .B .B .B .B .B .B .B
;   AGAIN ;
    head ZQUIT,5,ZQUIT,docolon
       DW lit,0
zquit1:  DW lit,0dh,EMIT,lit,0ah,EMIT,DUP,DOTA
       DW DOTB,DOTB,DOTB,DOTB,DOTB,DOTB,DOTB,DOTB
       DW DOTB,DOTB,DOTB,DOTB,DOTB,DOTB,DOTB,DOTB
       DW branch,zquit1
```

<br><sub>Last edited: 2022-02-04 22:23:50</sub>
