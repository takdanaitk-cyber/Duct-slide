# Duct Slide Rule (แอป Android)

ไม้บรรทัดสไลด์คำนวณท่อลม ใช้ได้ออฟไลน์ โปรเจกต์นี้ให้ GitHub สร้างไฟล์ APK ให้อัตโนมัติ ไม่ต้องลงโปรแกรมอะไรในคอมพิวเตอร์

## ขั้นตอน (ราว 10 นาที ทำบนคอมพิวเตอร์)

1. **สร้าง repository ใหม่** ที่ github.com กด New repository ตั้งชื่อ เช่น `duct-slide-rule` เลือก Private หรือ Public ก็ได้ แล้วกด Create repository
2. **อัปโหลดไฟล์** แตก zip นี้ก่อน แล้วในหน้า repository กด *uploading an existing file* ลากไฟล์และโฟลเดอร์ทั้งหมดในโฟลเดอร์ `duct-slide-rule-apk` เข้าไป (ต้องมีโฟลเดอร์ `www`, `resources`, `.github` และไฟล์ `package.json`, `capacitor.config.json`) แล้วกด Commit changes
   - โฟลเดอร์ `.github` ขึ้นต้นด้วยจุด บน Mac ต้องกด Cmd+Shift+. ใน Finder เพื่อแสดงไฟล์ที่ซ่อน
   - ถ้า `.github` ไม่ขึ้นหลังอัปโหลด ให้กด Add file > Create new file พิมพ์ชื่อไฟล์ `.github/workflows/build-apk.yml` แล้ววางเนื้อหาจากหัวข้อ "ไฟล์ workflow" ด้านล่าง
3. **รอสร้างแอป** เปิดแท็บ **Actions** จะเห็นงานชื่อ *Build Android APK* กำลังทำงาน (ประมาณ 5–10 นาที) ถ้าไม่เริ่มเอง กดงานนั้นแล้วเลือก Run workflow
4. **ดาวน์โหลด APK** เมื่อขึ้นเครื่องหมายถูกสีเขียว กดเข้าไปในงานนั้น เลื่อนลงไปที่ **Artifacts** กด `duct-slide-rule-apk` จะได้ไฟล์ zip แตกออกมาได้ `app-debug.apk`
5. **ติดตั้งบนมือถือ** ส่ง `app-debug.apk` เข้าเครื่อง (Google Drive, LINE Keep, สาย USB) แล้วแตะไฟล์เพื่อติดตั้ง
   - ครั้งแรก Android จะขอให้อนุญาตการติดตั้งแอปจากแหล่งนี้ กดอนุญาต
   - Play Protect อาจเตือนว่าเป็นแอปที่ไม่รู้จัก เพราะไฟล์นี้เซ็นด้วยคีย์ทดสอบ เลือก Install anyway (ติดตั้งต่อ)

## แก้ไขแอปภายหลัง

แก้ไฟล์ `www/index.html` บน GitHub (กดรูปดินสอ) แล้ว Commit GitHub จะสร้าง APK ใหม่ให้อีกรอบ ค่าที่ตั้งไว้ในแอปจะยังอยู่หลังติดตั้งทับ ถ้าเป็นแอปเดิมจากชุดเดียวกัน

## ถ้าสร้างไม่ผ่าน

เปิดงานที่ขึ้นเครื่องหมายแดงในแท็บ Actions กดขั้นตอนที่มีเครื่องหมาย x แล้วคัดลอกข้อความช่วงท้ายส่งให้ Claude ดู

## ไฟล์ workflow (`.github/workflows/build-apk.yml`)

```yaml
name: Build Android APK

on:
  push:
    branches: [main, master]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: Install dependencies
        run: npm install

      - name: Create Android project
        run: npx cap add android

      - name: Apply app icon and colors
        run: cp -r resources/res/. android/app/src/main/res/

      - name: Copy web files into the app
        run: npx cap sync android

      - name: Build APK
        working-directory: android
        run: ./gradlew assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: duct-slide-rule-apk
          path: android/app/build/outputs/apk/debug/app-debug.apk
          if-no-files-found: error
```

## หมายเหตุ

- สมการ: V = 183.35·Q/D², Δp = 0.109136·Q^1.9/D^5.02 (in.wg ต่อ 100 ft) สำหรับท่อเหล็กอาบสังกะสีและอากาศมาตรฐาน ไม่รวมความดันตกของ fitting และอุปกรณ์
- ใช้ Capacitor 6.2.3 ห่อหน้าเว็บเป็นแอป ฟอนต์ฝังอยู่ในโฟลเดอร์ `www/fonts` จึงไม่ต้องใช้อินเทอร์เน็ต
