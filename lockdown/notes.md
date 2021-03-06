
## The Lockdown

Reckless hacking has its consequences. You have locked yourself out of your car, the multimedia screen is not working and engine does not start. After some tests you find out that it can be unlocked back with a password, but you don't know it. You can either take the car to a service station for a costly repair, or figure some other way.

## Initial Analysis:
```
Chip locked
Please write your password: my face hurts
ACCESS DENIED
Chip locked
Please write your password: your password
ACCESS DENIED
Chip locked
```

## Reverse Engineering
Addresses
	2000
	1ffc
	2142
	--
	2142, 
	230b
```python
avr_loader_emu(0x1ffc, 0x2000, 0x2142)
avr_bss_emu(0x2142,0x230b)
```

BinDiff with Benzinegate hit all but four subs, this is going to be easy ;)

## Disassembly
```c
const short FiveA = 0x5a5a
const short AFive = 0xa5a5

char array_1021ee[128];
char read_buffer_10226e[129]

void main(void) {
	// stack 0x13 bytes
	char[14] y1;
	short    y5a;    // Y+0xf..0x10
	short    ya5;    // Y+0x11..0x12
	char     yCheck; // Y+0x13

	flag_set_mask(0);
	config_usart(USARTC0_DATA_ptr_ptr)
	init_flag_array();
	50 * NOP
	randomize_21ee_13b();
	50 * NOP
	randomize_21ee_13b();
	50 * NOP
	randomize_21ee_13b();
	50 * NOP

	while (true) {
		yCheck = 0xd;
		y5a = 5a;
		ya5 = a5;
		usart_print(USARTC0_DATA_ptr_ptr, "Chip locked\nPlease write your password: ");
		zero_and_read_str_165(USARTC0_DATA_ptr_ptr);

		yCheck = yCheck*2 + 3
		if ( diff_buffs_183(&word_10226e, &array_1021ee, 0x80) != 0) {
			usart_print(USARTC0_DATA_ptr_ptr, "ACCESS DENIED\n");
		} else {
			y5a = AFive;
			ya5 = FiveA;
		}
		if (y5a == FiveA && ya5 == AFive) continue;
		if (yCheck != 0x1d) {
			usart_print(USARTC0_DATA_ptr_ptr, "You are not supposed to be here..\n");
			continue;
		}

		50 * NOP
		yCheck = yCheck*2 +3;
		50 * NOP
		if ( diff_buffs_183(&word_10226e, &array_1021ee, 0x80) != 0) {
			usart_print(USARTC0_DATA_ptr_ptr, "ILLEGAL ACTIVITY DETECTED, ACCESS DENIED");
			continue;
		}
		usart_print(USARTC0_DATA_ptr_ptr, "Chip unlocked\n");
		usart_print(USARTC0_DATA_ptr_ptr, "Your flag is: ");
		if (yCheck != 0x3d) {
			usart_print(USARTC0_DATA_ptr_ptr, "What did I tell you!? Get out!");
			continue;
		}

		yCheck = yCheck*2 +3;
		flag_set_mask(0xff);
		if (yCheck != 0x7d) {
			usart_print(USARTC0_DATA_ptr_ptr, "... This is your final warning, stop it.\n");
			continue;
		}

		yCheck = yCheck*2 +3;
		print_flag(&usartc0_send_byte);
		if (yCheck != 0x7d) {
			usart_print(USARTC0_DATA_ptr_ptr, "Wait, how did you get all the way over here?\n");
			continue;
		}

		yCheck = ((yCheck*2 +3) & 0xf) - 1;
		flag_set_mask(0);
		if (yCheck != 0x7d) {
			usart_print(USARTC0_DATA_ptr_ptr, "Hah, you went too far!\n");
			continue;
		}

		break;
	}
}

void randomize_21ee_13b() {
	for (int8_t y1 = 0; y1 >= 0; y++) { // WHO WRITES CODE LIKE THIS!!!
		r24 = (char) random_dword();
		array_1021ee[i] = r24;
	}
	array_1021ee[y1-1]='\0';
}

void zero_and_read_str_165(USART_t *usart) {
	memset(read_str_until, 0, 128);
	read_str_until(usart, read_buffer_10226e, 128, 'a');
}

diff_buffs_183(char *buf1, char *buf2, char buflen) {
	char y1 = 0;
	char y2 = 0;
	char* y3 = buf1;
	char* y5 = buf2;
	char y7 = buflen;
	for (y1 = 0; y1 < buflen; y1++) {
		y2 |= buf1[y1] ^ buf2[y1];
	}
	return y2;
}

```

## Exploit Hunting

Option 1:
	* glitch out all three calls to `randomize_21ee_13b()`
	* hope its initialized to 0

Option 2:
	* branch jam after "ACCESS DENIED"
	* fairly large window...
	* we get feedback
	AND THEN
	* ride the nop sled
	* wait for "ILLEGAL ACTIVITY DETECTED, ACCESS DENIED"
	* clock-skip the rjmp into "Chip unlocked"
	* get flag

Option 3:
	* large scale clock skipping

