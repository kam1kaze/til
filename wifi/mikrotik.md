http://pezz.tkwcy.ee/mikrotik.txt

## Install password-less access via SSH
```
HOST=192.168.88.1

# Get necessary pub key and upload it to the router
pub_auth=$(ssh-add -l | awk '{print $3 ".pub";exit}')
lftp -u admin $HOST -e "put $pub_auth; ls; exit"

# Install such key
ssh admin@${HOST} "/user ssh-keys import public-key-file=${pub_auth##*/}; /user ssh-keys print"
```

## Software upgrade

### Pre-configuration 
```
# set software channel to stable (long-term, stable, testing, development)
/system package update set channel=stable

# set firmware/bootloader upgrade to auto, so it does not need another button click, but still two reboots
/system routerboard settings set auto-upgrade=yes silent-boot=no
```

### Install upgrades
```
# check if there's new software for selected channel
/system package update check-for-updates

# install latest software from selected channel and also reboot
/system package update install

# check that current-firmware=upgrade-firmware and same version as packages
/system routerboard print

# if current-firmware != upgrade-firmware
/system routerboard upgrade

# reboot again to install updated firmware/bootloader
/system reboot
```

## Security
```
# disable PMKID to add security
/interface wireless security-profiles set disable-pmkid=yes default
```

