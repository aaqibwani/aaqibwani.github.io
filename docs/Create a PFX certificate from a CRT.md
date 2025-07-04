---
title: Create a PFX certificate from a CRT
nav_order: 5
---
# Create a PFX certificate from a CRT and private key using Windows

1. Create a new folder and place your .crt and .key file in it. _If the private key file is a .txt file, change extension to .key._

2. Rename both files to have the same name (but different extension):

{{certificate_name}}.crt

{{certificate_name}}.key

Example:

 mail.crt --> CRT file
 
 mail.key --> private key

3. Open Command Prompt and navigate to the folder and run:

            certutil -mergepfx mail.crt mail.pfx


{: .important }
> You will be asked to create a password for the .pfx file. Store it as it will be required during certificate import on the target machine.

![Screenshot 2024-09-08 021252](https://github.com/user-attachments/assets/0ed1fb7c-561c-4cc8-a482-d8a98f99e890)






