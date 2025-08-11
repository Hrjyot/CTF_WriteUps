
# 🔐 CTF Write-Up – Brute-Forcing a Partially Known BitLocker Recovery Key

**Category:** Forensics  
**CTF Name:** Why2025
**Author:** Harry 
**Date:** 2025-08-10

---

## 📜 Overview
In this challenge, we were given an `SD_card.001` disk image from a CTF.  
Upon inspection, it was found to contain a **BitLocker-encrypted partition**.  
We were also provided with a **partially covered BitLocker recovery key** — the last 5 digits were obscured.

By using GPU acceleration under **WSL2** with an NVIDIA GPU, we brute-forced the missing digits and unlocked the volume to recover the flag.

> **Note:** The Flag is redacted to avoid spoiling the challenges from this CTF.

---

## 🗂 Files Provided
- `SD_card.001` (3.7 GB raw disk image)
- Partial recovery key:  718894-682847-228371-253055-328559-381458-030668-0XXXXX

- ## 🛠 Step-by-Step Solution

### 1️⃣ Identify Partition Layout
We checked the disk image structure:

fdisk -l SD_card.001
Found one partition starting at sector 8192.

Partition was marked FAT32 but actually contained BitLocker (-FVE-FS- signature).

2️⃣ Extract the Partition
Calculated the offset:
8192 × 512 = 4194304 bytes

Extracted the partition:
dd if=SD_card.001 of=bitlocker_partition.img bs=512 skip=8192

4️⃣ Extract the Recovery Hash

mkdir ~/bitcracker_hashes
./build/bitcracker_hash -i bitlocker_partition.img -o ~/bitcracker_hashes
We got:

Copy
hash_recv_pass.txt

5️⃣ Generate Candidate Keys

for i in $(seq -w 0 99999); do
  echo "718894-682847-228371-253055-328559-381458-030668-0${i}"
done > keys.txt
This created 100,000 possible keys.

6️⃣ Brute-Force with Bitcracker_CUDA

./build/bitcracker_cuda -r \
  -f ~/bitcracker_hashes/hash_recv_pass.txt \
  -d keys.txt

Result:

Password found: 718894-682847-228371-253055-328559-381458-030668-047839

7️⃣ Unlock the Volume

sudo mkdir -p /mnt/bitlocker /mnt/mount
sudo dislocker -V bitlocker_partition.img \
  --recovery-password=718894-682847-228371-253055-328559-381458-030668-047839 \
  -- /mnt/bitlocker
sudo mount -o loop /mnt/bitlocker/dislocker-file /mnt/mount

8️⃣ Retrieve the Flag

grep -r -n "flag{" /mnt/mount


