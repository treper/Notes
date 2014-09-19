#####Shell Notes

* scp带端口:

		scp -P 80 -r dir xx@host:/home/xx/

* 清空缓存:

		sudo sh -c "echo 3 > / proc/sys/vm/drop_caches"

* 查看系统版本号

		1. cat /etc/redhat-release
		2. rpm -q centos-release
		3. uname -r
		4. uname -a

