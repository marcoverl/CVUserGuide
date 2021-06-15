..    include:: <isonum.txt>

Frequently Asked Questions
==========================


-  I get this error when accessing via SSH my virtual machine:

   ::

       @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                                                                 
       @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @                                                                 
       @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                                                                 
       Permissions 0644 for '/home/user/my_key.pem' are too open.                                                                  
       It is required that your private key files are NOT accessible by others.                                                    
       This private key will be ignored.                                                                                           
       Load key "/home/user/my_key.pem": bad permissions                                                                           
       ubuntu@10.64.14.12: Permission denied (publickey).  


   SOLUTION: the IP of your VM has probably been reused. You need to
   update the relevant entry on your */home/user/.ssh/known\_hosts* file.
   Please run:

   ::

       ssh-keygen -R 10.64.51.7                                                                                                    

   and try again.



-----------------------------------------------------


