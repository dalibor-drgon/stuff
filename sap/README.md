# Externe interrupty na joystick na atmegal169

Datasheet DS: https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7735-Automotive-Microcontrollers-ATmega169P_Datasheet.pdf

*Ulohy boli navrhnute tak, aby sa dali implementovat aj bez externych interruptov. Je vsak mozne externe interrupty pouzit. To, ci je to dobry napad alebo nie, je otazne.*

Na povolenie externych interruptov - rovnako ako aj pre ine periferie - je potreba konfiguracia periferii pred pouzitim (casto len pri starte)
a implementacia handlera.

Podla kapitoly `11. External interrupts` DS je vidiet, ze nas budu zaujimat `PCI0` a `PCI1` interrupty. Je tam tiez vidiet, ktore `PCINTx` spadaju pod ktory `PCIy` interrupt handler. Na zaciatku kapitoly `1. Pin Configurations` je mozne vidiet, ktore piny patria pod ktore PCINTx flagy. Napr. `E0` patri pod `PCINT0`, `E1` pod `PCINT1`, ..., `B0` pod `PCINT8`, `B1` pod `PCINT9` atd az po `PCINT15`.

Na strane 48 DS v sekci `Interrupts` je tabulka ukazujuca adresy flash pamate, na ktore je pri interrupte skakane. Nas zaujima PCINT0 a PCINT1:
```
0x0004              jmp    PCINT0       ; PCINT0 Handler
0x0006              jmp    PCINT1       ; PCINT1 Handler
```

Na tieto adresy flash pamate (flash = sekcia kodu) je teda treba zapisat instrukciu `jmp` na dany neskor implementovany handler.

### Inicializacia

#### IO ports

Dalej nas bude zaujimat inicializacia PCIx interruptov. Na to nam posluzi kapitola 12. `I/O ports` a zvlast podsekcia `Register Description`.
Podla tabulky `12-1` je vidiet, ze ak je flag `DDxn` v registri `DDRy` nastaveny na nulu (co je defaultna hodnota), tak je brany ako vstup. Cize nemusime s tymto registrom nic menit. Tiez chceme, aby `PORTx = 0`, co je tiez defaultna hodnota, takze ani v tomto registri nie je treba nic menit.

#### External interrupts

Podkapitola `11.2 Register Description`. Nastavime `PCIE1` a `PCIE0` pin change interrupt enable bity na `1`. Dalej je treba povolit pozadovane `PCINTx` interrupty v `PCMSKy` registroch. Vsimnime si `EIFR` register - ten uklada interrupty flagy pre jednotlive `PCIx`. Tento flag je automaticky nastaveny na nulu po exekucii handleru, ktory implementujeme neskor.

#### Global interrupts

Nezabudnite povolit globalne interrupty cez instrukciu `sei`.

### Handler

Nakoniec je treba implementovat `PCINT0` a `PCINT1` handler. My to vsak zjednodusime na jeden handler. Tejto handler musi okrem zalohy pouzitych registrov a SREG registra urobit citanie z `E` a/alebo `B` portu a podla toho vykonat nejaku akciu (napr. nastavit vyhradeny register/globalnu premennu na aktualny stav registra). Aj ked to nie je optimalne a considered 'good-practice`, v jednoduchych programoch je mozne implementovat anti-ghosting priamo v tomto handleri - napr. precita sa port, pocka sa par tisic cyklov, znova sa precita port, hodnoty sa porovnaju a nastavi sa vyhradeny register/globalna premenna na precitanu hodnotu.

```asm
PCINT0:
PCINT1:
    push r16
    in r16, SREG
    push r16
    ; .... do some stuff
    pop r16
    out SREG, r16
    pop r16
    reti
```
