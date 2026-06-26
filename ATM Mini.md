.MODEL SMALL
.STACK 200H
.DATA
    ; ---------------------------------------------------------------------
    ; TAMPILAN ANTARMUKA / UI STRINGS
    ; ---------------------------------------------------------------------
    HEADER  DB 13,10,'====================================================='
            DB 13,10,'* SISTEM ATM MINI ANNISA AL FAJRIYA *'
            DB 13,10,'=====================================================$'
            
    GARIS   DB 13,10,'-----------------------------------------------------$'
    
    L_PIN   DB 13,10,'[+] MASUKKAN 4 DIGIT PIN ANDA: $'
    L_ERR   DB 13,10,'[!] PIN YANG ANDA MASUKKAN SALAH!$'
    L_BLCK  DB 13,10,'[!!!] KARTU ANDA DIBLOKIR! 3 KALI SALAH PIN. [!!!]$'
    
    MENU    DB 13,10,' '
            DB 13,10,'============ MENU UTAMA TRANSAKSI ATM ============='
            DB 13,10,'1. INFORMASI CEK SALDO'
            DB 13,10,'2. PENARIKAN TUNAI (PAKET NOMINAL)'
            DB 13,10,'3. PENARIKAN TUNAI MANDIRI (NOMINAL BEBAS)'
            DB 13,10,'4. SETORAN TUNAI'
            DB 13,10,'5. TRANSFER DANA KE REKENING LAIN'
            DB 13,10,'6. FITUR UBAH PIN ATM'
            DB 13,10,'7. LIHAT RIWAYAT MUTASI TERAKHIR'
            DB 13,10,'8. KELUAR DARI SISTEM'
            DB 13,10,'-----------------------------------------------------'
            DB 13,10,'MASUKKAN PILIHAN MENU ANDA (1-8): $'

    T_MENU  DB 13,10,' '
            DB 13,10,'============== MENU PENARIKAN TUNAI ================'
            DB 13,10,'SILAHKAN PILIH PAKET NOMINAL PENARIKAN:'
            DB 13,10,'-----------------------------------------------------'
            DB 13,10,'1. RP 50.000'
            DB 13,10,'2. RP 100.000'
            DB 13,10,'3. RP 200.000'
            DB 13,10,'4. RP 500.000'
            DB 13,10,'5. RP 1.000.000'
            DB 13,10,'-----------------------------------------------------'
            DB 13,10,'PILIH NOMINAL (1-5): $'

    T_CUSTOM DB 13,10,' '
             DB 13,10,'=========== PENARIKAN NOMINAL BEBAS ============'
             DB 13,10,'* KELIPATAN RP 50.000 (CONTOH: KETIK 150 UNTUK 150.000)'
             DB 13,10,'Masukkan Jumlah Ribuan (Tanpa .000): $'
    E_CUST   DB 13,10,'[!] JUMLAH HARUS KELIPATAN 50 ATAU DI ATAS NOL!$'

    S_MENU  DB 13,10,' '
            DB 13,10,'================ MENU SETOR TUNAI =================='
            DB 13,10,'SILAHKAN PILIH NOMINAL SETORAN ANDA:'
            DB 13,10,'-----------------------------------------------------'
            DB 13,10,'1. RP 50.000'
            DB 13,10,'2. RP 100.000'
            DB 13,10,'3. RP 200.000'
            DB 13,10,'4. RP 500.000'
            DB 13,10,'-----------------------------------------------------'
            DB 13,10,'PILIH NOMINAL SETORAN (1-4): $'

    TF_MENU DB 13,10,' '
            DB 13,10,'============== MENU TRANSFER REKENING =============='
            DB 13,10,'1. RP 50.000'
            DB 13,10,'2. RP 100.000'
            DB 13,10,'3. RP 200.000'
            DB 13,10,'4. RP 500.000'
            DB 13,10,'-----------------------------------------------------'
            DB 13,10,'PILIH NOMINAL DANA TRANSFER (1-4): $'

    P_MENU  DB 13,10,' '
            DB 13,10,'================== MENU UBAH PIN ==================='
            DB 13,10,'MASUKKAN PIN LAMA ANDA: $'
    P_NEW   DB 13,10,'MASUKKAN PIN BARU ANDA (4 DIGIT): $'
    P_SUCC  DB 13,10,'[+] PIN ATM BERHASIL DIPERBARUI! $'

    IN_REK  DB 13,10,'MASUKKAN NOMOR REKENING TUJUAN (4 DIGIT): $'
    S_MSG   DB 13,10,'[i] TOTAL SALDO REKENING ANDA : RP $'
    T_SUCC  DB 13,10,'[+] PENARIKAN TUNAI BERHASIL DIPROSES!$'
    T_FAIL  DB 13,10,'[-] TRANSAKSI GAGAL! SALDO REKENING ANDA TIDAK MENCUKUPI !!$'
    SET_SUC DB 13,10,'[+] SETORAN TUNAI BERHASIL! SALDO OTOMATIS BERTAMBAH.$'
    TF_SUCC DB 13,10,'[+] TRANSFER SUKSES! DANA BERHASIL DIKIRIM KE REK TUJUAN.$'
    
    M_MSG   DB 13,10,' '
            DB 13,10,'=============== RIWAYAT MUTASI TERAKHIR ============='
            DB 13,10,'LOG TRANSAKSI TERAKHIR: $'
    M_NONE  DB 13,10,'[i] BELUM ADA AKTIVITAS TRANSAKSI DI SESI INI.$'
    
    E_MSG   DB 13,10,' '
            DB 13,10,'====================================================='
            DB 13,10,'* TERIMA KASIH ATAS TRANSAKSI ANDA         * '
            DB 13,10,'=====================================================$'
    ERR_MSG DB 13,10,'[!] PILIHAN TIDAK VALID! SILAHKAN PILIH ULANG.$'

    STR_TARIK  DB 'PENARIKAN TUNAI   : RP $'
    STR_SETOR  DB 'SETORAN TUNAI     : RP $'
    STR_TF     DB 'TRANSFER KE REK   : $'
    STR_TF2    DB ' SEBESAR RP $'
    RUPIAH_NOL DB '.000$'
    KEMBALI_M  DB 13,10,'TEKAN TOMBOL APAPUN UNTUK KEMBALI KE MENU...$'

    ; ---------------------------------------------------------------------
    ; VARIABLES
    ; ---------------------------------------------------------------------
    SALDO       DW 40         ; Base Unit 1 = Rp 25.000, Maka 40 = Rp 1.000.000
    TERAKHIR    DW 0          
    HAS_TRANS   DB 0          ; 0=Kosong, 1=Tarik, 2=Setor, 3=Transfer
    REK_TUJUAN  DB '----$'    
    
    PIN_SISTEM  DB '1','2','3','4' ; PIN default awal
    PIN_INPUT   DB 4 DUP(?)        
    PIN_BARU    DB 4 DUP(?)        
    ATTEMPTS    DB 0               
    
    KUST_BUF    DB 6, 0, 6 DUP(0)  
    KUST_VAL    DW 0

.CODE
START:
    MOV AX, @DATA
    MOV DS, AX

    LEA DX, HEADER
    MOV AH, 09H
    INT 21H

; =========================================================================
; AUTENTIKASI PIN LOGIN
; =========================================================================
LOGIN_SISTEM:
    LEA DX, GARIS
    MOV AH, 09H
    INT 21H
    
    LEA DX, L_PIN
    MOV AH, 09H
    INT 21H
    
    MOV CX, 4
    XOR SI, SI
LOOP_PIN_INPUT:
    MOV AH, 07H
    INT 21H
    MOV PIN_INPUT[SI], AL
    
    ; Efek masking password bintang '*'
    MOV DL, '*'
    MOV AH, 02H
    INT 21H
    INC SI
    LOOP LOOP_PIN_INPUT
    
    XOR SI, SI
    MOV CX, 4
CEK_PIN_LOOP:
    MOV AL, PIN_INPUT[SI]
    CMP AL, PIN_SISTEM[SI]
    JNE PIN_SALAH
    INC SI
    LOOP CEK_PIN_LOOP
    JMP LOGIN_SUKSES

PIN_SALAH:
    INC ATTEMPTS
    LEA DX, L_ERR
    MOV AH, 09H
    INT 21H
    
    MOV AL, ATTEMPTS
    CMP AL, 3
    JGE KARTU_BLOCKED
    JMP LOGIN_SISTEM

KARTU_BLOCKED:
    LEA DX, L_BLCK
    MOV AH, 09H
    INT 21H
    JMP KELUAR

LOGIN_SUKSES:
    LEA DX, GARIS
    MOV AH, 09H
    INT 21H

; =========================================================================
; LOOP CORE MENU UTAMA
; =========================================================================
UTAMA:
    LEA DX, MENU
    MOV AH, 09H
    INT 21H
    
    MOV AH, 01H
    INT 21H
    MOV BL, AL

    CMP BL, '1'
    JE JMP_CEK
    CMP BL, '2'
    JE JMP_TARIK
    CMP BL, '3'
    JE JMP_CUSTOM
    CMP BL, '4'
    JE JMP_SETOR
    CMP BL, '5'
    JE JMP_TF
    CMP BL, '6'
    JE JMP_PIN
    CMP BL, '7'
    JE JMP_MUTASI
    CMP BL, '8'
    JE JMP_KELUAR
    
    LEA DX, ERR_MSG
    MOV AH, 09H
    INT 21H
    JMP UTAMA

; Jumper Jauh Mengatasi Limitasi Range JMP x86
JMP_CEK:    JMP PROSES_CEK
JMP_TARIK:  JMP MENU_TARIK
JMP_CUSTOM: JMP MENU_CUSTOM_TARIK
JMP_SETOR:  JMP MENU_SETOR
JMP_TF:     JMP MENU_TRANSFER
JMP_PIN:    JMP MENU_UBAH_PIN
JMP_MUTASI: JMP PROSES_MUTASI
JMP_KELUAR: JMP KELUAR

; =========================================================================
; MENU 1: CEK SALDO
; =========================================================================
PROSES_CEK:
    LEA DX, S_MSG
    MOV AH, 09H
    INT 21H
    MOV AX, SALDO
    CALL TAMPIL_SALDO
    CALL HIT 
    JMP UTAMA

; =========================================================================
; MENU 2: PENARIKAN PAKET
; =========================================================================
MENU_TARIK:
    LEA DX, T_MENU
    MOV AH, 09H
    INT 21H
    
    MOV AH, 01H
    INT 21H
    CMP AL, '1'
    JE T_50
    CMP AL, '2'
    JE T_100
    CMP AL, '3'
    JE T_200
    CMP AL, '4'
    JE T_500
    CMP AL, '5'
    JE T_1000
    
    LEA DX, ERR_MSG
    MOV AH, 09H
    INT 21H
    JMP MENU_TARIK

T_50:   MOV CX, 2   ; Base Unit Rp25.000: (50.000/25.000 = 2)
        JMP EKSEKUSI_TARIK
T_100:  MOV CX, 4   
        JMP EKSEKUSI_TARIK
T_200:  MOV CX, 8   
        JMP EKSEKUSI_TARIK
T_500:  MOV CX, 20  
        JMP EKSEKUSI_TARIK
T_1000: MOV CX, 40

EKSEKUSI_TARIK:
    MOV AX, SALDO
    CMP AX, CX
    JL GAGAL_TRANSAKSI
    SUB AX, CX
    MOV SALDO, AX
    MOV TERAKHIR, CX
    MOV HAS_TRANS, 1    
    LEA DX, T_SUCC
    MOV AH, 09H
    INT 21H
    CALL HIT
    JMP UTAMA

GAGAL_TRANSAKSI:
    LEA DX, T_FAIL
    MOV AH, 09H
    INT 21H
    CALL HIT 
    JMP UTAMA

; =========================================================================
; MENU 3: PENARIKAN CUSTOM (NOMINAL BEBAS)
; =========================================================================
MENU_CUSTOM_TARIK:
    LEA DX, T_CUSTOM
    MOV AH, 09H
    INT 21H
    
    LEA DX, KUST_BUF
    MOV AH, 0AH
    INT 21H
    
    ; Parsing String ASCII Ke Integer
    XOR AX, AX
    XOR CX, CX
    MOV CL, KUST_BUF[1]  
    JCXZ ERR_CUSTOM
    LEA SI, KUST_BUF[2]
    MOV BX, 10

LOOP_CONVERT:
    MOV DX, 0
    MUL BX              
    XOR DX, DX
    MOV DL, [SI]
    SUB DL, '0'         
    CMP DL, 9
    JA ERR_CUSTOM       
    ADD AX, DX
    INC SI
    LOOP LOOP_CONVERT
    
    ; Cek apakah kelipatan 50.000
    MOV BX, 50
    XOR DX, DX
    DIV BX
    CMP DX, 0
    JNE ERR_CUSTOM      
    
    SHL AX, 1           ; Hitung kebutuhan unit saldo (*2)
    CMP AX, 0
    JE ERR_CUSTOM
    
    MOV CX, AX          
    JMP EKSEKUSI_TARIK

ERR_CUSTOM:
    LEA DX, E_CUST
    MOV AH, 09H
    INT 21H
    CALL HIT
    JMP UTAMA

; =========================================================================
; MENU 4: SETOR TUNAI
; =========================================================================
MENU_SETOR:
    LEA DX, S_MENU
    MOV AH, 09H
    INT 21H
    
    MOV AH, 01H
    INT 21H
    CMP AL, '1'
    JE S_50
    CMP AL, '2'
    JE S_100
    CMP AL, '3'
    JE S_200
    CMP AL, '4'
    JE S_500
    
    LEA DX, ERR_MSG
    MOV AH, 09H
    INT 21H
    JMP MENU_SETOR

S_50:   MOV CX, 2
        JMP EKSEKUSI_SETOR
S_100:  MOV CX, 4
        JMP EKSEKUSI_SETOR
S_200:  MOV CX, 8
        JMP EKSEKUSI_SETOR
S_500:  MOV CX, 20

EKSEKUSI_SETOR:
    MOV AX, SALDO
    ADD AX, CX          
    MOV SALDO, AX
    MOV TERAKHIR, CX
    MOV HAS_TRANS, 2    
    LEA DX, SET_SUC
    MOV AH, 09H
    INT 21H
    CALL HIT
    JMP UTAMA

; =========================================================================
; MENU 5: TRANSFER
; =========================================================================
MENU_TRANSFER:
    LEA DX, IN_REK
    MOV AH, 09H
    INT 21H
    
    MOV CX, 4
    XOR SI, SI
LOOP_REK:
    MOV AH, 01H
    INT 21H
    MOV REK_TUJUAN[SI], AL
    INC SI
    LOOP LOOP_REK
    
    LEA DX, TF_MENU
    MOV AH, 09H
    INT 21H
    
    MOV AH, 01H
    INT 21H
    CMP AL, '1'
    JE TF_50
    CMP AL, '2'
    JE TF_100
    CMP AL, '3'
    JE TF_200
    CMP AL, '4'
    JE TF_500
    
    LEA DX, ERR_MSG
    MOV AH, 09H
    INT 21H
    JMP MENU_TRANSFER

TF_50:  MOV CX, 2
        JMP EKSEKUSI_TF
TF_100: MOV CX, 4
        JMP EKSEKUSI_TF
TF_200: MOV CX, 8
        JMP EKSEKUSI_TF
TF_500: MOV CX, 20

EKSEKUSI_TF:
    MOV AX, SALDO
    CMP AX, CX
    JL GAGAL_TRANSAKSI_TF  
    SUB AX, CX          
    MOV SALDO, AX
    MOV TERAKHIR, CX
    MOV HAS_TRANS, 3    
    LEA DX, TF_SUCC
    MOV AH, 09H
    INT 21H
    CALL HIT
    JMP UTAMA

GAGAL_TRANSAKSI_TF:
    JMP GAGAL_TRANSAKSI

; =========================================================================
; MENU 6: UBAH PIN
; =========================================================================
MENU_UBAH_PIN:
    LEA DX, P_MENU
    MOV AH, 09H
    INT 21H
    
    MOV CX, 4
    XOR SI, SI
LOOP_REPIN:
    MOV AH, 07H
    INT 21H
    MOV PIN_INPUT[SI], AL
    MOV DL, '*'
    MOV AH, 02H
    INT 21H
    INC SI
    LOOP LOOP_REPIN
    
    XOR SI, SI
    MOV CX, 4
CEK_REPIN:
    MOV AL, PIN_INPUT[SI]
    CMP AL, PIN_SISTEM[SI]
    JNE PIN_UBAH_SALAH
    INC SI
    LOOP CEK_REPIN
    
    LEA DX, P_NEW
    MOV AH, 09H
    INT 21H
    
    MOV CX, 4
    XOR SI, SI
LOOP_NEWPIN:
    MOV AH, 07H
    INT 21H
    MOV PIN_SISTEM[SI], AL 
    MOV DL, '*'
    MOV AH, 02H
    INT 21H
    INC SI
    LOOP LOOP_NEWPIN
    
    LEA DX, P_SUCC
    MOV AH, 09H
    INT 21H
    CALL HIT
    JMP UTAMA

PIN_UBAH_SALAH:
    LEA DX, L_ERR
    MOV AH, 09H
    INT 21H
    CALL HIT 
    JMP UTAMA

; =========================================================================
; MENU 7: RIWAYAT MUTASI
; =========================================================================
PROSES_MUTASI:
    CMP HAS_TRANS, 0
    JE MUTASI_KOSONG
    
    LEA DX, M_MSG
    MOV AH, 09H
    INT 21H
    
    CMP HAS_TRANS, 1
    JE MUTASI_TARIK
    CMP HAS_TRANS, 2
    JE MUTASI_SETOR
    
    LEA DX, STR_TF
    MOV AH, 09H
    INT 21H
    LEA DX, REK_TUJUAN  
    MOV AH, 09H
    INT 21H
    LEA DX, STR_TF2
    MOV AH, 09H
    INT 21H
    JMP PRINT_MUTASI_VAL

MUTASI_TARIK:
    LEA DX, STR_TARIK
    MOV AH, 09H
    INT 21H
    JMP PRINT_MUTASI_VAL

MUTASI_SETOR:
    LEA DX, STR_SETOR
    MOV AH, 09H
    INT 21H

PRINT_MUTASI_VAL:
    MOV AX, TERAKHIR
    CALL TAMPIL_SALDO
    CALL HIT
    JMP UTAMA

MUTASI_KOSONG:
    LEA DX, M_NONE
    MOV AH, 09H
    INT 21H
    CALL HIT 
    JMP UTAMA

; =========================================================================
; MENU 8: KELUAR
; =========================================================================
KELUAR:
    LEA DX, E_MSG
    MOV AH, 09H
    INT 21H
    MOV AH, 4CH         ; Interupsi DOS untuk menutup program keluar [cite: 20]
    INT 21H

; ---------------------------------------------------------------------
; PROCEDURES & SUBROUTINES
; ---------------------------------------------------------------------
HIT PROC NEAR
    LEA DX, KEMBALI_M
    MOV AH, 09H
    INT 21H
    MOV AH, 07H
    INT 21H
    RET
HIT ENDP

TAMPIL_SALDO PROC
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    
    CMP AX, 0
    JNE HITUNG
    MOV DL, '0'
    MOV AH, 02H
    INT 21H
    JMP SELESAI_PRINT

HITUNG:
    MOV BX, 25          ; Skala base unit Rp25.000
    MUL BX              
    MOV CX, 0
    MOV BX, 10
LOOP_BAGI:
    MOV DX, 0
    DIV BX
    PUSH DX             
    INC CX
    CMP AX, 0
    JNE LOOP_BAGI
LOOP_PRINT:
    POP DX              
    ADD DL, '0'         
    MOV AH, 02H
    INT 21H
    LOOP LOOP_PRINT

    LEA DX, RUPIAH_NOL
    MOV AH, 09H
    INT 21H
SELESAI_PRINT:
    POP DX
    POP CX
    POP BX
    POP AX
    RET
TAMPIL_SALDO ENDP

END START
