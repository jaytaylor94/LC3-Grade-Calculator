; LC‑3 Program: Grade Statistics Calculator 
; Author: Jonathan Taylor and Jose Joya
; Course: CIS‑11  ▸ Project Part 2
; ------------------------------------------------------------
;  Prompts for five test‑scores (0‑100), then prints:
;    • Minimum  • Maximum  • Average (integer)  • Letter grade
;  Satisfies rubric: subroutines, stack, save‑restore, branching,
;  overflow‑safe constants, ASCII↔︎integer, pointers, etc.
; ------------------------------------------------------------
;  Assemble & Run in PennSim / LC‑3Edit:
;     1.  assemble   2. reset   3. run   4. type scores + <Enter>
; ------------------------------------------------------------

        .ORIG x3000
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;  Program‑level set‑up
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
        LD   R6, SP_INIT        ; R6 = stack pointer (x4000)
        LEA  R2, SCORES         ; R2 → array base
        LD   R3, FIVE           ; loop counter = 5
        AND  R4, R4, #0         ; R4 = running SUM = 0

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;  INPUT LOOP – read scores, store, accumulate SUM
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
INPUT_LOOP
        JSR  GET_SCORE          ; ↳ score in R0 (0‑100)
        STR  R0, R2, #0         ; SCORES[i] = score
        ADD  R4, R4, R0         ; SUM  += score
        ADD  R2, R2, #1         ; ++array pointer
        ADD  R3, R3, #-1        ; --counter
        BRp  INPUT_LOOP

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;  POST‑PROCESSING
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
        JSR  FIND_MIN_MAX       ; sets MIN / MAX

        ; ---- Compute AVG = SUM / 5 (R4 holds SUM) ----
        LD   R3, FIVE           ; divisor = 5
        AND  R5, R5, #0         ; quotient  = 0
DIV5_LOOP
        ADD  R4, R4, #-5        ; SUM -= 5
        BRn  DIV5_DONE
        ADD  R5, R5, #1         ; ++quotient
        BR   DIV5_LOOP
DIV5_DONE
        ST   R5, AVG            ; AVG = quotient

        JSR  MAP_GRADE          ; derive GRADE letter
        JSR  DISPLAY_ALL        ; print results
        HALT

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; ---------------- SUBROUTINES ----------------
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; -----------------------------------------------------------
;  GET_SCORE  – prompt / read multi‑digit decimal (0‑100)
;  Returns score in R0   (clobbers R1‑R5)
; -----------------------------------------------------------
GET_SCORE
        ; PUSH R7 -----------------------------------------
        ADD  R6, R6, #-1
        STR  R7, R6, #0

        ; Prompt text
        LEA  R0, PROMPT
        TRAP x22                 ; Puts string

        AND  R1, R1, #0          ; R1 = accumulated value
READ_CHAR
        TRAP x23                 ; GETC  → R0
        LD   R2, ASCII_NL        ; newline constant
        NOT  R2, R2
        ADD  R2, R2, #1          ; two‑s complement of NL (‑NL)
        ADD  R2, R0, R2          ; R0 - NL
        BRz  END_INPUT           ; if newline → done

        ; Convert ASCII digit → binary -------------------
        LD   R3, ASCII_0
        NOT  R3, R3
        ADD  R3, R3, #1          ; two‑s complement of '0' (‑48)
        ADD  R0, R0, R3          ; R0 = digit 0‑9

        ; R1 = R1*10 + digit  (×8 + ×2)
        ADD  R4, R1, R1          ; tmp2 = R1*2
        ADD  R5, R4, R4          ; tmp4 = *4
        ADD  R5, R5, R5          ; tmp8 = *8
        ADD  R1, R4, R5          ; R1*10
        ADD  R1, R1, R0          ; + current digit
        BR   READ_CHAR
END_INPUT
        ADD  R0, R1, #0          ; return value in R0

        ; POP R7 ------------------------------------------
        LDR  R7, R6, #0
        ADD  R6, R6, #1
        RET

; -----------------------------------------------------------
;  FIND_MIN_MAX – scans SCORES[5] to set MIN & MAX
; -----------------------------------------------------------
FIND_MIN_MAX
        ADD  R6, R6, #-1         ; PUSH R7
        STR  R7, R6, #0

        LEA  R2, SCORES
        LDR  R0, R2, #0          ; first element
        ST   R0, MIN
        ST   R0, MAX
        LD   R3, FIVE            ; counter = 5
FMM_LOOP
        LDR  R0, R2, #0
        ; ---- MIN check ----
        LD   R1, MIN
        NOT  R1, R1
        ADD  R1, R1, #1          ; -MIN
        ADD  R1, R0, R1          ; R0 - MIN
        BRn  UPDATE_MIN
        BR   SKIP_MIN
UPDATE_MIN
        ST   R0, MIN
SKIP_MIN
        ; ---- MAX check ----
        LD   R1, MAX
        NOT  R1, R1
        ADD  R1, R1, #1          ; -MAX
        ADD  R1, R0, R1          ; R0 - MAX
        BRzp SKIP_MAX            ; if R0 ≤ MAX skip
        ST   R0, MAX
SKIP_MAX
        ADD  R2, R2, #1          ; ++pointer
        ADD  R3, R3, #-1
        BRp  FMM_LOOP

        ; POP R7
        LDR  R7, R6, #0
        ADD  R6, R6, #1
        RET

; -----------------------------------------------------------
;  MAP_GRADE – uses AVG → sets GRADE (ASCII letter)
; -----------------------------------------------------------
MAP_GRADE
        ADD  R6, R6, #-1         ; PUSH R7
        STR  R7, R6, #0

        LD   R0, AVG
        LD   R1, NINETY
        NOT  R1, R1
        ADD  R1, R1, #1          ; -90
        ADD  R1, R0, R1          ; AVG - 90
        BRn  SET_A
        LD   R1, EIGHTY
        NOT  R1, R1
        ADD  R1, R1, #1          ; -80
        ADD  R1, R0, R1
        BRn  SET_B
        LD   R1, SEVENTY
        NOT  R1, R1
        ADD  R1, R1, #1          ; -70
        ADD  R1, R0, R1
        BRn  SET_C
        LD   R1, SIXTY
        NOT  R1, R1
        ADD  R1, R1, #1          ; -60
        ADD  R1, R0, R1
        BRn  SET_D
        LD   R0, ASCII_F
        BR   STORE_GRADE
SET_D
        LD   R0, ASCII_D
        BR   STORE_GRADE
SET_C
        LD   R0, ASCII_C
        BR   STORE_GRADE
SET_B
        LD   R0, ASCII_B
        BR   STORE_GRADE
SET_A
        LD   R0, ASCII_A
STORE_GRADE
        ST   R0, GRADE
        ; POP R7
        LDR  R7, R6, #0
        ADD  R6, R6, #1
        RET

; -----------------------------------------------------------
;  PRINT_NUM – decimal 0‑100 (R0) → console
; -----------------------------------------------------------
PRINT_NUM
        ; PUSH R7,R1‑R3 -----------------------------------
        ADD  R6, R6, #-1
        STR  R7, R6, #0
        ADD  R6, R6, #-1
        STR  R1, R6, #0
        ADD  R6, R6, #-1
        STR  R2, R6, #0
        ADD  R6, R6, #-1
        STR  R3, R6, #0

        ; --- check for 100 ---
        LD   R1, NEG_100
        ADD  R2, R0, R1        ; R2 = val - 100
        BRz  PRINT_100         ; =0 → value is 100
        BRn  LESS_THAN_100     ; <0 → <100, continue
; (val >100 should never occur)

PRINT_100
        LD   R3, ASCII_1
        TRAP x21
        LD   R3, ASCII_0
        TRAP x21
        TRAP x21
        BR   NUM_DONE

LESS_THAN_100
        ; tens & ones -----------------------------------
        AND  R1, R1, #0         ; tens = 0
TEN_LOOP
        ADD  R0, R0, #-10
        BRn  ONES_STAGE
        ADD  R1, R1, #1         ; ++tens
        BR   TEN_LOOP
ONES_STAGE
        ADD  R0, R0, #10        ; remainder
        ; print tens if >0
        ADD  R2, R1, #0
        BRz  PRINT_ONES
        LD   R3, ASCII_0
        ADD  R3, R3, R1
        TRAP x21
PRINT_ONES
        LD   R3, ASCII_0
        ADD  R3, R3, R0
        TRAP x21

NUM_DONE
        ; restore ---------------------------------------
        LDR  R3, R6, #0
        ADD  R6, R6, #1
        LDR  R2, R6, #0
        ADD  R6, R6, #1
        LDR  R1, R6, #0
        ADD  R6, R6, #1
        LDR  R7, R6, #0
        ADD  R6, R6, #1
        RET

; -----------------------------------------------------------
;  DISPLAY_ALL – prints labels + values
; -----------------------------------------------------------
DISPLAY_ALL
        ADD  R6, R6, #-1        ; PUSH R7
        STR  R7, R6, #0

        ; Min
        LEA  R0, STR_MIN
        TRAP x22
        LD   R0, MIN
        JSR  PRINT_NUM

        ; Max
        LEA  R0, STR_MAX
        TRAP x22
        LD   R0, MAX
        JSR  PRINT_NUM

        ; Avg
        LEA  R0, STR_AVG
        TRAP x22
        LD   R0, AVG
        JSR  PRINT_NUM

        ; Grade
        LEA  R0, STR_GRADE
        TRAP x22
        LD   R0, GRADE
        TRAP x21

        ; newline
        LD   R0, ASCII_NL
        TRAP x21

        ; POP R7
        LDR  R7, R6, #0
        ADD  R6, R6, #1
        RET

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; ---------------- CONSTANTS & DATA ----------------
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
PROMPT      .STRINGZ "\nEnter score: "
STR_MIN     .STRINGZ "\nMin: "
STR_MAX     .STRINGZ "\nMax: "
STR_AVG     .STRINGZ "\nAvg: "
STR_GRADE   .STRINGZ "\nGrade: "

; numeric constants
FIVE        .FILL #5
NEG_100     .FILL #-100

; grade thresholds
SIXTY       .FILL #60
SEVENTY     .FILL #70
EIGHTY      .FILL #80
NINETY      .FILL #90

; ASCII codes
ASCII_0     .FILL x0030
ASCII_1     .FILL x0031
ASCII_NL    .FILL x000A
ASCII_A     .FILL x0041
ASCII_B     .FILL x0042
ASCII_C     .FILL x0043
ASCII_D     .FILL x0044
ASCII_F     .FILL x0046

; storage
SCORES      .BLKW 5
MIN         .BLKW 1
MAX         .BLKW 1
AVG         .BLKW 1
GRADE       .BLKW 1

; stack pointer init
SP_INIT     .FILL x4000

        .END
