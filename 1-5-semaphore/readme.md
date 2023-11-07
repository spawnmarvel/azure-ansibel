# Semaphore UI

https://www.ansible-semaphore.com/

# Install


https://www.ansible-semaphore.com/blog/installation-and-removal/

```bash

sudo -s

apt update

apt install snapd
# snapd is already the newest version (2.58+22.04.1).

snap install semaphore
# snap "semaphore" is already installed, see 'snap help refresh'

snap stop semaphore

# add user

snap start semaphore

# Open Semaphore UI in browser by address http://server456:3000


```