example make Pisound work with Stock Radxa Rockpi4b Stretch linux image.

Kernel 4.4.154 

Download image and follow the base steps below 
(also work with radxa linux image latest (4.4.154-110 atm))
	Essential Steps - Starting point for build kernel modules after reboot:
		-------------------------------------------------

		https://wiki.radxa.com/Rockpi4/radxa-apt

		export DISTRO=stretch-testing	
		echo "deb http://apt.radxa.com/$DISTRO/ ${DISTRO%-*} main" | sudo tee -a /etc/apt/sources.list.d/apt-radxa-com.list
		wget -O - apt.radxa.com/$DISTRO/public.key | sudo apt-key add -	
		sudo apt-get update && sudo apt-get upgrade -y	
		sudo apt-get install -y rockchip-overlay rockchip-fstab	

		sudo reboot
		-------------------------------------------------

sudo apt-get install -y libssl-dev build-essential git
wget https://github.com/radxa/kernel/archive/4.4.154-90-rockchip.tar.gz

tar -xvf 4.4.154-90-rockchip.tar.gz
cd kernel-4.4.154-90-rockchip 

uname -r > version.txt 
export ARCH=arm64
			
make -j5 rockchip_linux_defconfig

export lv=-$(cut -d- -f2,3,4 /home/linaro/kernel-4.4.154-90-rockchip/version.txt)
export kv=$(make -j5 kernelversion) 
			
make EXTRAVERSION=$lv -j5  modules_prepare scripts prepare modules_prepare
cd ..
			
sudo mv kernel-4.4.154-90-rockchip /usr/src/linux
sudo unlink /lib/modules/$(uname -r)/build
sudo unlink /lib/modules/$(uname -r)/source
sudo ln -s /usr/src/linux /lib/modules/$(uname -r)/build
sudo ln -s /usr/src/linux /lib/modules/$(uname -r)/source
		
cd ~

git clone -b rockpi4 https://github.com/verplant23/pisound.git

cd pisound/pisound-module

make -j4 ARCH=arm64
ARCH=arm64 sudo -E make -j4 install
