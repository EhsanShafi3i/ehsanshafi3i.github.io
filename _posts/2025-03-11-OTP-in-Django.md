---
title: OTP in Django Without Saving it in the Database (Using Redis)
image: 
    path: /assets/posts/otp-in-django.jpg
    comment: true
tags: [python,django,redis]
---


I searched a lot but couldn’t find the ideal way to implement OTP without storing it in a database. So, I will show my own approach.
---
> ## `WARNING: This is my opinion, it could be wrong or vulnerable`
---

## 1. Installation

## Install Required Packages

**Recommendation:** Use the uv package manager instead of pip.

### Installing uv:
```bash
    pip install uv 
```
### Create a Virtual Environment and Activate It:
```bash
    uv venv
    .venv\Scripts\activate  # Windows
    source .venv/bin/activate  # macOS/Linux
```
### Install Required Packages:
```bash
    uv pip install django-redis
```
## 2. Redis & Docker Configuration

Go to settings.py and add Redis configuration for caching:
```python
    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://cache:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient",
                "PASSWORD": REDIS_PASSWORD,
            },
        }
    }
```
## Configure Redis Password in docker-compose.yml
```yml
      cache:
        image: redis:6.2-alpine
        restart: always
        ports:
          - '6379:6379'
        enviroment:
          - REDIS_PASSWORD=$REDIS_PASSWORD
        command: redis-server --save 20 1 --loglevel warning --requirepass ${REDIS_PASSWORD}
        volumes: 
          - cache:/data
```
## 3. OTP Generator and Handler

Create otp_handler.py in your utils directory and add the following code:
```python
    import hashlib
    import hmac
    import random 
    
    from django.conf import settings
    from django.core.cache import cache
    
    
    class OTPHandler:
        """Generate OTP an verify OTP."""
    
        PHONE_SECRET_KEY = settings.PHONE_SECRET_KEY
    
        @staticmethod
        def _get_hashed_phone(phone_number):
            """Hash the number."""
            return hmac.new(
                OTPHandler.PHONE_SECRET_KEY.encode(), phone_number.encode(), hashlib.sha256
            ).hexdigest()
    
        @staticmethod
        def generate_otp(phone_number):
            """Generate Five-digit code and save it in cache."""
            hashed_phone = OTPHandler._get_hashed_phone(phone_number)
            if cache.get(f"otp_{hashed_phone}"):
                return {
                        "status": "pending", 
                        "message": "OTP already sent. Please wait."
                        }
            otp = str(random.randint(10000,99999)
    
            cache.set(f"otp_{hashed_phone}", otp, timeout=120)
    
            return otp
    
        @staticmethod
        def verify_otp(phone_number, otp_code):
            """Get number and OTP and verify it, then delete it from cache."""
            hashed_phone = OTPHandler._get_hashed_phone(phone_number)
            stored_otp = cache.get(f"otp_{hashed_phone}")
            cache.add(f"otp_attempts_{hashed_phone}", 0, timeout=120)
    
            attempts = cache.incr(f"otp_attempts_{hashed_phone}")
    
            if attempts > 3:
                return {
                    "status": "blocked",
                    "message": "Too many attempts. Try again later.",
                }
    
            if stored_otp:
                if stored_otp == otp_code:
                    cache.delete(f"otp_{hashed_phone}")
                    cache.delete(f"otp_attempts_{hashed_phone}")
                    return {"status": "valid", "message": "OTP Verified"}
    
                return {"status": "invalid", "message": "Invalid OTP"}
    
            return {
                "status": "expired",
                "message": "Your OTP has been expired, request for new OTP",
            }
```
## Explanation

The OTPHandler class consists of three main parts:

### **1. _get_hashed_phone(phone_number)**:

* Hashes the phone number using PHONE_SECRET_KEY to enhance security.

### **2. generate_otp(phone_number):**

* Hashes the phone number.

* Generates a five-digit OTP.

* Stores the OTP in the cache for **120 seconds (2 minutes)**.

### **3. verify_otp(phone_number, otp_code)**:

* Retrieves the stored OTP from the cache and compares it with the provided OTP.

* Limits OTP attempts to **3 retries**.

* If successful, deletes the OTP and resets the attempt counter.

* Returns a status indicating whether the OTP is **valid, invalid, expired, or blocked**.

**Note:** Instead of hashing the phone number, you can store it in plain text if needed.
>  I hope it helps
