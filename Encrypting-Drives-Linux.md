## INTRODUCTION
This epic Michael Guide is here to help when encrypting new drives! Be careful and make sure you follow all steps correctly particularly around fstab!

# First we need to create the luksFormat for the drive(s) and then open them under a new partition
sudo cryptsetup luksFormat /dev/(drive-name)
sudo cryptsetup open /dev/(drive-name) (partition-name-wanted)

# Now we need to mark it as ext4 and mount the drive
sudo mkfs.ext4 /dev/mapper/(partition-name)
sudo mount /dev/mapper/(partition-name) (mount-locaton(e.g. "/mnt/game1"))

# Now we need to navigate to the cryptsetup-keys.d directory
sudo ls /etc/cryptsetup-keys.d

# And here we create the keys
sudo nano /etc/cryptsetup-keys.d/(keyname).key

# Remove the empty line space at the end of the key files using perl
sudo perl -pi -e 'chomp if eof' /etc/cryptsetup-keys.d/(keyname).key

# Now we add the key to the drive
sudo cryptsetup luksAddKey /dev/(drive-name) /etc/cryptsetup-keys.d/(keyname).key

# (OPTIONAL) You can now test to see if this works!
sudo umount (mount-locaton(e.g. "/mnt/game1"))
sudo cryptsetup close (partition-name)
sudo cryptsetup open /dev/(drive-name) (partition-name) --key-file /etc/cryptsetup-keys.d/(keyname).key

## FSTAB MOMENT BE VERRYYYY CAREFUL! ONLY ADD THE LINES INTENDED
# Open fstab in nano (this file is checked every startup and here we will setup the drives to automatically mount them)
sudo nano /etc/fstab

(ADD THESE LINES TO FSTAB BELOW WHAT IS ALREADY PRESENT)
/dev/mapper/(partition-name) (mount-locaton(e.g. "/mnt/game1")) ext4 defaults,nofail 0 0
/dev/mapper/(partition-name) (mount-locaton(e.g. "/mnt/game1")) ext4 defaults,nofail 0 0

# Make sure the drive is open (decrypted)
sudo cryptsetup open /dev/(drive-name) (partition-name) --key-file /etc/cryptsetup-keys.d/(keyname).key

# Find the UUID of the drive
sudo blkid /dev/(drive-name)

( MAKE A NOTE OF THIS UUID SOMEWHERE E.G: /dev/sdb1: UUID="6c4eb3a5-8b30-411e-9bf3-389947bcd6e1" )

# Read the crypttab file (to ensure it exists)
sudo cat /etc/crypttab

# Alter the file to include the drives with the UUID and key
sudo nano /etc/crypttab

( ENTER INTO THE FILE )
(partition-name) UUID=(uuid) /etc/cryptsetup-keys.d/(keyname).key

# Make sure changes were saved!
sudo cat /etc/crypttab

# Reboot

systemctl reboot

or

reboot

or 

sudo reboot

# You can see your new partitions!
df -h

#### HOPE IT WORKED!!!