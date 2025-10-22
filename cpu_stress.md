# Ubuntu EC2 CPU Stress Test Guide

This guide explains how to perform a CPU stress test on an **Ubuntu EC2 instance** to verify **Auto Scaling** and monitor CPU utilization.

---

## 1️⃣ Connect to EC2

```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

## 2️⃣ Install Stress Tool

Install `stress-ng`, which allows specifying CPU load:

```bash
sudo apt update
sudo apt install stress-ng -y
```

* `stress-ng` is preferred for controlled CPU load.

---

## 3️⃣ Check Number of vCPUs

```bash
lscpu | grep "^CPU(s):"
```

* Example output: `CPU(s): 2` → 2 cores
* Use this number in stress-ng.

---

## 4️⃣ Run CPU Stress at 75% Load

Assuming 2 vCPUs:

```bash
stress-ng --cpu 2 --cpu-load 75 --timeout 10m
```

**Explanation:**

* `--cpu 2` → number of CPU workers (match your vCPUs)
* `--cpu-load 75` → target 75% CPU per worker
* `--timeout 10m` → run for 10 minutes (adjust as needed)

> You can adjust `--cpu-load` to simulate higher or lower load.

---

## 5️⃣ Monitor CPU Usage

Install and use `htop` to watch CPU:

```bash
sudo apt install htop -y
htop
```

* CPU usage should be around 75%
* Check CloudWatch → **EC2 → CPUUtilization** to see if Auto Scaling triggers.

---

## 6️⃣ Observe Auto Scaling

* Ensure your **Auto Scaling group (ASG)** is configured correctly (min/max instances, target CPU 70%)
* Watch **ALB Target Group** → new instances become healthy and receive traffic
* If CPU drops below threshold → ASG may terminate extra instances

---

## 7️⃣ Stop the Test Early (Optional)

```bash
pkill stress-ng
```

* Stops the stress test immediately if needed.

---

## 8️⃣ Tips & Notes

* Run this in **QA / non-production** only
* Make sure **Auto Scaling group min/max** allows scaling
* Duration (`--timeout`) and load (`--cpu-load`) can be adjusted for testing
* Observe CloudWatch metrics for proper scaling behavior

---

## ✅ Summary

* This test allows you to **simulate 75% CPU load**
* Check if **Auto Scaling group launches new instances** when CPU exceeds threshold
* Stop stress anytime with `pkill stress-ng`
* Use **htop** or **CloudWatch** to monitor CPU
