lsblk



sudo mount /dev/nvme1n1 /data



mkfs.xfs /dev/nvme1n1

- filesystem 확인하기 `sudo file -s /dev/nvme?n*`

```bash
/dev/nvme0n1:     x86 boot sector; partition 1: ID=0xee, starthead 0, startsector 1, 16777215 sectors, extended partition table (last)\011, code offset 0x63
/dev/nvme0n1p1:   SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
/dev/nvme0n1p128: data
/dev/nvme1n1:     SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
```







  192  sudo mkfs.xfs /dev/nvme1n1
  193  sudo mkfs.xfs -f /dev/nvme1n1
  194  lsblk
  195  history
  196   
  197  sudo growpart /dev/nvme0n1 1
  198  df -h
  199  df /h
  200  df -h
  201  history
  202  sudo mount /dev/nvme1n1 /data
  203  df -h





sudo du -sh /data