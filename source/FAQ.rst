..    include:: <isonum.txt>

Frequently Asked Questions
==========================

Account renewal and expiration
------------------------------

- I received an email with the following text:

   ::

    Your affiliation to the project XYZ is going to expire
    If you wish to renew the affiliation please open the CloudVeneto dashboard in a browser and select the project: XYZ.
    The renewal procedure will be proposed to you.

    Please, don't reply to this message

  What does this mean ? What should I do ?

  SOLUTION: if you are interested to renew your account, please follow the procedure described at 
  :ref:`Project membership renewal<ProjectMembershipRenewal>`. Your renewal request will then be evaluated by the
  project manager. 
  If you don't take any action, your account will be disabled (and eventually deleted).

--------------------------------

- I received an email with the following text:


   ::

    #######################################
    ##   I T A L I A N    V E R S I O N  ##
    #######################################

    Salve,
    a seguito della rimozione del suo account su CloudVeneto da tutti i
    progetti a cui era affiliato, l'accesso alla macchina frontend
    gate.cloudveneto.it mediante il suo account xyz e' stato
    disattivato.

    Cordiali saluti,

          Il supporto CloudVeneto

    #######################################
    ##   E N G L I S H    V E R S I O N  ##
    #######################################

    Hello,
    since your account on CloudVeneto has been disassociated from
    any active project, your access to the gate.cloudveneto.it machine
    through the xyz account has been disabled.

    Regards,

         The CloudVeneto support team

  What does this mean ? What should I do ?

  SOLUTION: When you aren't affiliated anymore to any project, your account on CloudVeneto gets automatically
  disabled (and it will be deleted after a while). 
  This happens if you didn't apply for the account renewal (see :ref:`Project membership 
  renewal<ProjectMembershipRenewal>`) or if your renewal request wasn't approved by the project manager.
  If you think your account should be reactivated, please 
  contact the admins at support@cloudveneto.it.


Problems accessing my virtual machine via SSH
-----------------------------------------------

-  I get this error when accessing via SSH my virtual machine:

   ::

       @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                         
       @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @                         
       @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                         
       IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!                               
       Someone could be eavesdropping on you right now (man-in-the-middle attack)!         
       It is also possible that the RSA host key has just been changed.                    
       The fingerprint for the RSA key sent by the remote host is                          
       c8:1b:1d:37:61:ee:9f:e4:db:5b:31:91:35:7b:d2:59.                                    
       Please contact your system administrator.                                           
       Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.     
       Offending key in /home/user/.ssh/known_hosts:3                                      
       RSA host key for 10.64.51.7 has changed and you have requested strict checking.     
       Host key verification failed.                                                       

   SOLUTION: the IP of your VM has probably been reused. You need to
   delete the offending entry on your */home/user/.ssh/known\_hosts* file.
   For example you can use:

   ::

       sed -i '/^10.64.51.7 /d' .ssh/known_hosts

   and try again to connect.

I can't see anymore my data
---------------------------

- I can't see anymore my data after a reboot of my instance 

  SOLUTION: Are the data on an volume ? Did you remember to mount this volume ? Please also see  :ref:`this section<RemountDisk>`.
  

-----------------------------------------------------

Problems with supplementary ephemeral storage
---------------------------------------------

- I am using a flavor which has a supplementary ephemeral disk, but I not able to change ownership / mode of a directory

  SOLUTION: A ‘fat’ file system is used for the supplementary ephemeral disk. We suggest to reformat this file system as described in
  :ref:`Flavors with supplementary ephemeral disk <FlavorsWithSupEpDisk>`.    
