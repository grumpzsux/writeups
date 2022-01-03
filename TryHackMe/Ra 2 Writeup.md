
# Ra 2 Writeup TryHackme (Hard).
#### Sergio Medeiros | Twitter @GRuMPzSux
![image](https://user-images.githubusercontent.com/80599694/147979402-90fbbffd-f072-4f9a-bdfc-f0584a80d3f4.png)


## Summary:
#### This is a Windows Box with a rated difficulty of "Hard" that can only be access via your TryHackMe paid subscription. This contains various stages of in-depth enumeration every step of the way including, password cracking, NTLMv2 hash capturing, exploiting with SweetPotato via Powershell Access and other fun yet challenging steps along the way!  I loved doing this box.

### Backstory:
![image](https://user-images.githubusercontent.com/80599694/147980047-20364107-8930-4965-a9b3-cbccde7c9ea6.png)

## Enumeration Phase
#### First thing is first, I decided to do a quick scan on the target with Nmap.
````bash
nmap -sV -sC -T4 -oA nmap/initial 10.10.213.222
````
![Image](https://user-images.githubusercontent.com/80599694/147980457-0791d10d-4ae6-47b9-a7ea-e258abd1d451.png)
