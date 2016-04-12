# emu8086-Pong-Game-2-players-
;Pong GAME-cu 2 jucatori
;Reguli-tasta r pt a acorda jucatorului rosu prima lovitura
;.......-tasta g pt a acorda jucatorului verde prima lovitura
;.......-space pt trimite mingea la celalalt jucator
;.......-fiecare jucator are doar doua vieti
;.......-scorul este afisat in partea de jos a ecranului  
;.......-pentru deplasare jucatorul rosu foloseste tastele:q(sus) si a(jos)
;.......-pentru deplasare jucatorul verde foloseste tastele:o(sus) si l(jos)


data segment
    ;Se introduc datele(dimensiunile) 
    
    red_player db 10,15 
    green_player db 10,15
    ball_old db 60,5   ; xtop_vechi, ytop_vechi, xbottom_vechi, ybottom_vechi
    ball_new db 60,5   ; xtop_nou, ytop_nou, xbottom_nou, ybottom_nou
    score db "Rosu 0 VS 0 Verde   $" ;afisare scor
    mov_option db 0
    ip db 0
    last_win db 0 
    pkey db "press any key...$"
    
ends
 
 
      
stack segment

dw   128  dup(0)

ends
code segment  
start:

; seteaza registrii

    mov ax, data
    mov ds, ax
    mov es, ax
    
    restart_game:
    
    mov green_player[0], 8 
    mov green_player[1], 13
    mov red_player[0], 8
    mov red_player[1], 13
    mov mov_option, 0
    mov ip, 0 
    
    
    ;Stergere ecran  
    Mov AH, 06h
    Mov AL, 00h
    Mov BH, 0011
    Mov CH, 00h
    Mov CL, 00h
    Mov DH, 025
    Mov Dl, 080
    int 10H

    ; setare cursor (0,0)
    mov bh, 0
    mov ah, 2
    mov dl, 6
    mov dh, 22  
    int 10h
    
    lea dx, score
    mov ah, 9
    int 21h        ;string returnat la ds:dx
    
    cmp score[6] ,"1" 
    je game_over
    cmp score[10],"1"
    je game_over
   
    
    call update_red
    call update_green
    
                               
        mov al, last_win[0]
        mov cx, 0 
        start_position:    
                
                cmp last_win[0], 0
                je set_start
                cmp last_win[0], "r"
                je set_red
                cmp last_win[0], "g"
                je set_green

                set_start:
                Mov Ah, 1H
                int 16H
                
                
                jz skipkeyboard2
                
                set_red:
                cmp al, "r"
                jne skip_red_start
                    mov ball_new[0],2
                    mov al, red_player[0]
                    add al,2
                    mov ball_new[1],al
                    call update_ball
                skip_red_start:
                
                set_green:
                cmp al, "g"
                jne skip_green_start
                    mov ball_new[0],76
                    mov al, green_player[0]
                    add al,2
                    mov ball_new[1],al
                    call update_ball    
                skip_green_start:    
                
                cmp al," "
                je break_start_position 
                
               Call Keyboard_Select
               
               skipkeyboard2:
               ;flush the keyboard buffer
                Mov Ah, 0Ch
                Mov Al, 0h
                int 21H

                mov last_win[0], 0
        loop start_position
    
    break_start_position:
    
    mov cx, 0
    mov al, 0  
    again:
    
    call update_mov_option
    
    cmp ip[0],1
    je restart_game


    ;Verifica intrarea din tastatura si incarca in reg Al
    Mov Ah, 1H
    int 16H
     
     
    ;nu exista intrare din tastatura-sare peste pentru a face bucla rapida        
    JZ skip_keyboard        
       
    Call Keyboard_Select


    skip_keyboard:
    

    ;flush the keyboard buffer
    Mov Ah, 0Ch
    Mov Al, 0h
    int 21H

    loop again
    
    game_over:

            
    lea dx, pkey
    mov ah, 9
    int 21h        ;string de iesire ds:dx
    
    ;asteapta o tasta....    
    mov ah, 1
    int 21h
    
    mov ax, 4c00h ; iesire catre SO
    int 21h    
ends
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
update_red Proc near
    pushA
    
    ;Wipe out1  
    Mov AH, 06h
    Mov AL, 00h
    Mov BH, 0000           ;atribut albastru
    Mov CL, 1              ; top x
    mov CH, 0              ; top y 
    Mov Dl, 1              ; bottom x
    Mov DH, red_player[0]  ; bottom y
    dec dh  
    int 10H 

    ;Wipe out2  
   ; Mov AH, 06h
   ; Mov AL, 00h
   ; Mov BH, 9Eh            ; atribut albastru
   ; Mov CL, 1              ; top x
    Mov CH, red_player[1]   ; top y
    inc ch 
   ; Mov Dl, 1              ; bottom x
    Mov DH, 80              ; bottom y 
    int 10H

    ;display new  
   ; Mov AH, 06h
   ; Mov AL, 00h
    Mov BH, 0CEh             ; atribut rosu
   ; Mov CL, 1  ;            ; top x       
    Mov CH, red_player[0]    ; top y 
   ; Mov Dl, 1               ; bottom x    
    Mov DH, red_player[1]    ; bottom y
    int 10H 
           
    popA
    ret
update_red Endp
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
update_green Proc near
    pushA
    
    ;Wipe out1  
    Mov AH, 06h
    Mov AL, 00h
    Mov BH, 0000            ; atribut albastru
    Mov CL, 77              ; top x
    mov CH, 0               ; top y 
    Mov Dl, 77              ; bottom x
    Mov DH, green_player[0] ; bottom y
    dec dh  
    int 10H 

    ;Wipe out2  
   ; Mov AH, 06h
   ; Mov AL, 00h
   ; Mov BH, 9Eh             ; atribut albastru
   ; Mov CL, 77              ; top x
    Mov CH, green_player[1]  ; top y
    inc ch 
   ; Mov Dl, 77              ; bottom x
    Mov DH, 77               ; bottom y 
    int 10H

    
    ;display new  
   ; Mov AH, 06h
   ; Mov AL, 00h
    Mov BH, 0AEh                ; atribut rosu
   ; Mov CL, 77  ;              ; top x       
    Mov CH, green_player[0]     ; top y 
   ; Mov Dl, 77                 ; bottom x    
    Mov DH, green_player[1]     ; bottom y
    int 10H 
           
    popA
    ret
update_green Endp
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;   
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
update_ball Proc near
    pushA
    
    ;Wipe out1  
    Mov AH, 06h
    Mov AL, 00h
    Mov BH, 0000                     ;atribut albastru
    Mov CL, ball_old[0]              ; top x
    mov CH, ball_old[1]              ; top y 
    Mov Dl, ball_old[0]              ; bottom x
    Mov DH, ball_old[1]              ; bottom y
  
    int 10H 
    
    ;display new  
   ; Mov AH, 06h
   ; Mov AL, 00h
    Mov BH, 0EAh                     ; atribut rosu
    Mov CL, ball_new[0]              ; top x
    mov CH, ball_new[1]              ; top y 
    Mov Dl, ball_new[0]              ; bottom x
    Mov DH, ball_new[1]              ; bottom y
     int 10H
     
    mov ball_old[0], Cl
    mov ball_old[1], Ch     
    popA
    ret
update_ball Endp
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

 Keyboard_Select Proc Near;;Preluare in AL si comparare cu "ABCDQ" 
                          ;;seteaza flag-ul."12345"
 Push Bx
 Push Dx
 Push CX
  
 Cmp AL, "Q"
 JE Set_red_up
 Cmp AL, "q"
 JE Set_red_up 
 
 Cmp AL, "A"
 JE Set_red_down 
 Cmp AL, "a"
 JE Set_red_down 
 
 Cmp AL, "o"
 JE Set_green_up
 
 Cmp AL, "l"
 JE Set_green_down
   
 Jmp Keyboard_Select_Done:
   
  Set_red_up:
    mov bl, red_player[0]
    cmp bl, 0
    je pass_red_inc 
        dec red_player[0]
        dec red_player[1]
        call update_red
    pass_red_inc:
  JMP Keyboard_Select_Done
 
  Set_red_down:
    mov bl, red_player[1]
    cmp bl, 23
    je pass_red_dec
        inc red_player[0]
        inc red_player[1]
        call update_red
    pass_red_dec: 
  JMP Keyboard_Select_Done
 
  Set_green_up:
    mov bl, green_player[0]
    cmp bl, 0
    je pass_green_inc 
        dec green_player[0]
        dec green_player[1]
        call update_green
    pass_green_inc:
  JMP Keyboard_Select_Done
 
  Set_green_down:
    mov bl, green_player[1]
    cmp bl, 23
    je pass_green_dec
        inc green_player[0]
        inc green_player[1]
        call update_green
    pass_green_dec: 
  JMP Keyboard_Select_Done
    
                                                   
 SetQ:
 mov ah, 4ch ; iesire catre sistemul de operare
 int 21h
        
        
 Keyboard_Select_Done:
 
 Pop CX
 PoP Dx
 Pop BX
 ret   
 Keyboard_Select EndP   
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;


update_mov_option proc near
pusha
    

    cmp ball_new[0], 2
    jne skip_red_player
        

        mov al,red_player[0] 
        cmp ball_new[1], al
        jne skip0r
        
            mov mov_option[0], 1            
        skip0r:
        
        inc al         
        cmp ball_new[1], al 
        jne skip1r
        
            mov mov_option[0], 1                  
        skip1r:
        
        inc al        
        cmp ball_new[1], al 
        jne skip2r
        
            mov mov_option[0], 2                  
        skip2r:
        
        inc al        
        cmp ball_new[1], al 
        jne skip3r
        
            mov mov_option[0], 2                
        skip3r:
        
        inc al        
        cmp ball_new[1], al 
        jne skip4r
        
            mov mov_option[0], 3                  
        skip4r:
        
        inc al        
        cmp ball_new[1], al 
        jne skip5r
        
            mov mov_option[0], 3                 
        skip5r:        

    skip_red_player:
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
    cmp ball_new[0], 76
    jne skip_green_player
        
        mov al, green_player[0]        
        cmp ball_new[1], al
        jne skip0g
        
            mov mov_option[0], 6           
        skip0g:
        
        inc al          
        cmp ball_new[1], al 
        jne skip1g
        
        
            mov mov_option[0], 6                  
        skip1g:
        
        inc al        
        cmp ball_new[1], al
        jne skip2g
                
            mov mov_option[0], 5
                  
        skip2g:
        
        inc al        
        cmp ball_new[1], al 
        jne skip3g
           
            mov mov_option[0], 5                 
        skip3g:
        
        inc al        
        cmp ball_new[1], al 
        jne skip4g
                
            mov mov_option[0], 4                 
        skip4g:
        
        inc al        
        cmp ball_new[1], al
        jne skip5g
        
            mov mov_option[0], 4                 
        skip5g:        
    skip_green_player:
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
    cmp ball_new[1], 0 ;;; comparatie cu marginea superioara
    jne bottom_border
    
        cmp mov_option[0], 1
        jne swap_top_6
        
            mov mov_option[0], 3
            jmp border_check_done
            
        swap_top_6:
            mov mov_option[0], 4
        
        jmp border_check_done
        
    bottom_border:
    cmp ball_new[1], 23
    jne border_check_done
    
        cmp mov_option[0], 3
        jne swap_bottom_4
        
            mov mov_option[0], 1

            jmp border_check_done
            
        swap_bottom_4:
            mov mov_option[0], 6

    border_check_done:
    
    
    cmp ball_new[0], 1
    jne skip1:
        inc score[6]
        mov last_win[0], "g"
        mov ip[0],1
    Skip1:
    
    cmp ball_new[0],78
    jne Skip2:
        mov last_win[0], "r"
        inc score[10]
        mov ip[0],1
    Skip2:
    
     
    
    cmp mov_option[0],1
    je mov1        
    cmp mov_option[0],2
    je mov2        
    cmp mov_option[0],3
    je mov3
    cmp mov_option[0],4
    je mov4
    cmp mov_option[0],5
    je mov5
    cmp mov_option[0],6
    je mov6
    jmp mov_done
    
    mov1:
        inc ball_new[0]
        dec ball_new[1]
        call update_ball
    jmp mov_done
    
    mov2:
        inc ball_new[0]  
        call update_ball  
        
    jmp mov_done
    
    mov3:
        inc ball_new[0]
        inc ball_new[1]
        call update_ball   
    jmp mov_done 
    
    mov4:
        dec ball_new[0]
        inc ball_new[1]
        call update_ball        
    jmp mov_done 
 
    mov5:
        
        dec ball_new[0]
        call update_ball       
    jmp mov_done
    
    mov6:
        dec ball_new[0]
        dec ball_new[1]
        call update_ball 
   
    mov_done:
    
    
    
      
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;; 
   
popa
ret
update_mov_option endp    
    
     
 
end start ; 
