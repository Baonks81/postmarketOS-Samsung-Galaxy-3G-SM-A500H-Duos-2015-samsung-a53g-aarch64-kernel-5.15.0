# postmarketOS-Samsung-Galaxy-3G-SM-A500H-Duos-2015-samsung-a53g-aarch64-kernel-5.17.0
pmOS on Samsung galaxy 3G SM-A500H Duos samsung-a53g

Reference Model on wiki:

https://en.m.wikipedia.org/wiki/Samsung_Galaxy_A5_(2015)



Bài này có 3 đề mục chính:

Tìm hiểu về cách hệ thống boot
Tìm hiểu về dts trong lk2nd và linux-panel-drivers
Tìm hiểu cách cài pmOS rootfs.img vào samsung-a53g



I. Cách thức hệ thống boot:



Bắt đầu nào, từ khi Google tạo phần cứng tương thích dùng nhân Linux để chạy Android, Google vẫn giữ fastboot để flash lại firmware nếu có trục trặc, lỗi hệ thống hoặc nâng cấp mới. Khi các hãng lấy về, họ chỉ giữ nguyên Android system, còn lại bỏ fastboot và cập nhật, thay đổi firmware bằng chính phần mềm của họ. Trong đó có Samsung, dùng Kites3 như iTunes để quản lý máy, user data, upgrade firmware. Về kỹ thuật thì lại có Odin chạy trên Windows dùng để reflash/update firmware. Cộng đồng nguồn mở đã tạo 1 công cụ gần giống Odin, chạy trên Linux, gọi là Heimdall (dùng để flash lk2nd)



II. Về lk2nd và linux-panel-drivers



Chắc ai cũng nghe qua Fuchsia OS của Google. Dựa trên nhân Zircon( little kernel từ Code Aurora), thì lk2nd chính là viết tắt của secondary little kernel (là bản fork của little kernel như Zircon), cộng đồng nguồn mở đã fork thành công và đang phát triển lk2nd trên nhiều thiết bị. Mục đích là đem fastboot về các thiết bị đã bị loại bỏ fastboot, mà chạy phần mềm chính hãng. Chỉ có người trong cuộc mới hiểu người trong kẹt



Đem fastboot quay lại nhằm flash boot.img của pmOS để load kernel và rootfs để có thể dùng Linux mobile trên thiết bị di động

https://github.com/msm8916-mainline/lk2nd



linux-panel-drivers chính là các dtb phát sinh khi cài lk2nd, có dòng panel ss_dsi dùng để hiển thị hình ảnh trên gpu chipset, hiện tại chỉ có 1 số model và variant được hỗ trợ, cần được thêm vào nhiều hơn

https://github.com/msm8916-mainline/linux-mdss-dsi-panel-driver-generator



Sơ đồ boot như sau:



Secondary little kernel(lk2nd) → [ initramfs + vmlinuz + config + extra (/pmOS_boot của pmOS) → rootfs + firmware + kernel modules (/pmOS_root giao dien Phosh của pmOS) ]



III. Cách cài postmarketOS vào 3G SM-A500H/DS samsung-a53g



https://wiki.postmarketos.org/wiki/Samsung_Galaxy_A5_2015_(samsung-a5ulte)



Hiện tại, trên trang chủ wiki của pmOS ngoài hỗ trợ community cho Samsung Galaxy A5 bản quốc tế SM-A500F và bản EU SM-A500FU (2 mã samsung-a5lte và samsung-a5ulte), đã hỗ trợ 3G SM-A500H/DS (samsung-a53g)  trong file lk2nd/dts/msm8916/msm8216-samsung-r08.dts. không cần phải chỉnh sửa gì thêm



Theo tìm hiểu thì dùng Qualcomm-msm8916 có 5 nhánh nhỏ



https://patchwork.ozlabs.org/project/devicetree-bindings/patch/1450371534-10923-20-git-send-email-mtitinger+renesas@baylibre.com/



qcom-id-msm8916     206     ← a5lte và a5ulte
qcom-id-msm8016     247
qcom-id-msm8216     248     ← a53g là dòng này
qcom-id-msm8116     249
qcom-id-msm8616     250



Cài Alpine Linux trên Virutalbox:

[MEDIA=youtube]1_bsycXrFcI[/MEDIA]



Upgrade lên bảng mới nhất và cài các tool



# sudo apk update && apk upgrade



# sudo apk add git python3 dtc libfdt py3-libfdt gcc-arm-none-eabi heimdall make



# git clone https://github.com/msm8916-mainline/lk2nd.git



# cd lk2nd



# sudo make TOOLCHAIN_PREFIX=arm-none-eabi- msm8916-secondary



Sau khi built xong sẽ có lk2nd.img trong lk2nd/build-msm8916-secondary



Upgrade lên ROM Android 6.0.1



Đưa máy về Download mode bằng nhấn tổ hợp phím Power + Volume down + Home, sau đó nhấn Volume up



Kết nối 3G SM-A500H/DS vào PC/Laptop, chọn USB là Samsung devices trên menu của Virtualbox



# sudo heimdall flash --BOOT lk2nd.img



Vì lk2nd đã quản lý boot nên không cần flash -boot.img nữa, tham khảo ở đây

https://wiki.postmarketos.org/wiki/Qualcomm_Snapdragon_410/412_(MSM8916)



Chọn mirror gần với khu vực đang ở theo hướng dẫn dưới đây:

https://wiki.postmarketos.org/wiki/Mirrors



# sudo apk add pmbootstrap



No need TravMurav branch because master support samsung-a5 now. Thanks to Minecrell and  TravMurav ^_^!



/***------------------------

Build samsung-a5.img kernel 5.14.0 từ TravMurav branch trên gitlab postmarketos



# pmbootstrap init



Choose: edge → samsung → a5lte → manline modem → y → user → phosh → nano → en_US.UTF-8 → sm-a500-phosh → y



Add Travmurav branch for a53g support:



# cd $(pmbootstrap config aports)



# git remote add TravMurav https://gitlab.com/TravMurav/pmaports.git



# git fetch TravMurav



# git checkout TravMurav/gt510lte



-----------------------------***/



# pmbootstrap init



Channel []: edge



Vendor []: samsung



Device codename []: a5



Kernel []: mainline-modem



Enable this package? (y/n) [y]: y



Username []: user



User interface []: phosh ← hoặc plasma-mobile thì đổi ở đây



Extra packages []: nano



Choose default locale installation [en_US.UTF-8]: en_US.UTF-8



Device hostname (short form, e.g ‘foo’) []: sm-a500h-phosh



Build outdate packages during ‘pmbootstrap install’? (y/n) [y]: y



# pmbootstrap status



Trước khi build image thì chroot vào cài các package về local để tạo máy ảo, khỏi phải download và tạo khi build image



# pmbootstrap chroot



~/# apk add abuild build-base ccache git devicepkg-dev mkbootimg postmarketos-base ccache-cross-symlinks gcc-aarch64 g++-aarch64 crossdirect ncurses-dev bash bc bison elfutils-dev flex gmp-dev installkernel linux-headers openssl-dev perl sed binutils-aarch64



# pmbootstrap --details-to-stdout install



*** SET LOGIN PASSWORD FOR: 'user' ***

(rootfs_samsung-a5) % passwd user

New password: 147147

Retype new password: 147147



# pmbootstrap export



Image ở thư mục này: /home/[username]/.local/var/pmbootstrap/chroot_native/home/pmos/rootfs/samsung-a5.img



Boot ở đây: /home/[username]/.local/var/pmbootstrap/chroot_rootfs_samsung_a5/boot



# sudo fastboot flash userdata samsung-a5.img



# sudo fastboot erase system



# sudo fastboot reboot



Để kích hoạt call/modem/mobile data. Cài thêm cmd:qmicli



# sudo apk upgrade



# sudo apk add cmd:qmicli



# sudo rc-service msm-modem-uim-selection start



Edit phoc.ini



# sudo apk add xorg-server



# sudo cvt 720 1280 58.6



# sudo nano /usr/share/phosh/phoc.ini



[output: DSI-1]

modeline = 74.50  720 768 840 960  1280 1283 1293 1326 -hsync +vsync

mode = 720x1280

scale = 2



Build image log: https://www.mediafire.com/file/h3r98udh1ndm5lr



Voila! Enjoy postmarketOS-Phosh trên 3G SM-A500H/DS



Nếu upgrade bị khởi động lại phosh, thì phải kết nối USB Network/USB Internet qua SSH để upgrade



# sudo rc-service sshd start



Tham khảo USB Network: https://wiki.postmarketos.org/wiki/USB_Network



USB Internet: https://wiki.postmarketos.org/wiki/USB_Internet



# sudo apk update && apk upgrade



Full sources:

https://drive.google.com/folderview?id=1GVVLiGVojGf074vAvZoboN7e7cZE-Asi



Hoặc download trên trang chủ của postmarketOS

https://images.postmarketos.org/bpo/edge/samsung-a5/phosh/



default user is user, passwd is 147147



If you want to mount sparse image working with fastboot, you have to de-sparse it to working with gparted. Methods is img2simg/simg2img



# sudo apk gparted img2simg simg2img



# sudo simg2img samsung-a5.img samsung-a5-de-sparse.img



# sudo modprobe loop



# sudo losetup -f



# sudo losetup /dev/loop0 /path/to/samsung-a5-de-sparse.img



# sudo partprobe /dev/loop0



# sudo gparted /dev/loop0



Unmount image:



# sudo losetup -d /dev/loop0



Drop unallocated disk for ext4, here is example:



# sudo fdisk -l /path/to/samsung-a5-de-sparse.img



Disk samsung-a5-de-sparse.img: 6144 MB, 6144000000 bytes, 12000000 sectors

Units = sectors of 1 * 512 = 512 bytes

Sector size (logical/physical): 512 bytes / 512 bytes

I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk identifier: 0x000ea37d

Device Boot                  Start         End         Blocks      Id        System

samsung-a5-de-sparse.img            2048     9181183     4589568    b  W95 FAT32



truncate --size=$[(9181183+1)*512] /path/to/samsung-a5-de-sparse.img



https://askubuntu.com/questions/1174487/re-size-the-img-for-smaller-sd-card-how-to-shrink-a-bootable-sd-card-image#1174509



Drop unallocated disk for f2fs

https://wiki.ubuntu.com/Touch/Deploying



Sparse image to working with Android fastboot



# sudo img2simg samsung-a5-de-sparse.img samsung-a5-sparse.simg



# sudo mv samsung-a5-sparse.simg samsung-a5-sparse.img



We're on matrix:

https://app.element.io/#/room/#main:postmarketos.org

https://app.element.io/#/room/#devel:postmarketos.org

https://app.element.io/#/room/#lowlevel:postmarketos.org

https://app.element.io/#/room/#mainline:postmarketos.org

https://app.element.io/#/room/#porting:postmarketos.org



[MEDIA=youtube]zhkdnFGy29c[/MEDIA]



[MEDIA=youtube]XfJ5JPHQr1c[/MEDIA]



[MEDIA=youtube]Yo9FFAENrGs[/MEDIA]
