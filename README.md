ল্যান কেবল সংযোগ দিলে ওয়াই-ফাই অটোমেটিক ডিজেবল হওয়ার সমস্যা ও সমাধান
সমস্যার বিবরণ
ল্যান (Ethernet) কেবল সংযোগ দেওয়ার পর ওয়াই-ফাই অ্যাডাপ্টার স্বয়ংক্রিয়ভাবে নিষ্ক্রিয় (disable) হয়ে যায় এবং ম্যানুয়ালি চালু করা যায় না।

সমস্যার সম্ভাব্য কারণসমূহ
BIOS/UEFI-তে LAN/WLAN Auto Switching সক্রিয় থাকা

HP ডিভাইসের ক্ষেত্রে HP Switching Service সক্রিয় থাকা

নেটওয়ার্ক অ্যাডাপ্টারের পাওয়ার ম্যানেজমেন্ট সেটিংস

ডুয়াল নেটওয়ার্ক ইন্টারফেস কনফ্লিক্ট

সমাধানের ধাপসমূহ
ধাপ ১: BIOS/UEFI সেটিংস চেক করুন
markdown
1. কম্পিউটার রিস্টার্ট করুন
2. বুট হওয়ার সময় F2/F10/F12/Del কী চেপে BIOS-এ যান
3. Advanced বা Device Configuration মেনু খুঁজুন
4. LAN/WLAN Auto Switching অপশন খুঁজুন
5. যদি Enabled থাকে, তাহলে Disabled করুন
6. Save & Exit করুন
ধাপ ২: HP Switching Service ডিজেবল করুন (শুধুমাত্র HP ব্যবহারকারীদের জন্য)
powershell
# PowerShell অ্যাডমিনিস্ট্রেটর হিসেবে খুলুন
# নিচের কমান্ডগুলো চালান

# HP Switching Service স্টপ করুন
Stop-Service "HPDSUWatcherService" -ErrorAction SilentlyContinue
Stop-Service "HPLanWlanWwanSwitchingService" -ErrorAction SilentlyContinue

# সার্ভিস ডিজেবল করুন
Set-Service "HPDSUWatcherService" -StartupType Disabled -ErrorAction SilentlyContinue
Set-Service "HPLanWlanWwanSwitchingService" -StartupType Disabled -ErrorAction SilentlyContinue
ধাপ ৩: পাওয়ার ম্যানেজমেন্ট সেটিংস কনফিগার করুন
markdown
1. Device Manager খুলুন (Win + X → Device Manager)
2. Network adapters প্রসারিত করুন
3. Wi-Fi অ্যাডাপ্টারে ডান-ক্লিক → Properties
4. Power Management ট্যাবে যান
5. "Allow the computer to turn off this device to save power" এর টিক চিহ্ন তুলে দিন
6. একইভাবে LAN অ্যাডাপ্টারের জন্যও করুন
7. OK ক্লিক করুন
ধাপ ৪: রেজিস্ট্রি এডিট করে ফিক্স করুন (Advanced)
markdown
⚠️ সতর্কতা: রেজিস্ট্রি এডিট করার আগে ব্যাকআপ রাখুন
reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\0001]
"PnPCapabilities"=dword:00000024

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\0002]
"PnPCapabilities"=dword:00000024
ধাপ ৫: নেটওয়ার্ক রিসেট করুন
markdown
1. Settings খুলুন (Win + I)
2. Network & Internet → Advanced network settings
3. Network reset-এ ক্লিক করুন
4. Reset now বাটনে ক্লিক করুন
5. কম্পিউটার রিস্টার্ট করুন
ধাপ ৬: কমান্ড প্রম্পট দিয়ে ফিক্স করুন
cmd
# অ্যাডমিনিস্ট্রেটর হিসেবে CMD খুলুন
# নিচের কমান্ডগুলো একে একে চালান

# নেটওয়ার্ক কনফিগারেশন রিসেট করুন
netsh int ip reset
netsh winsock reset
ipconfig /release
ipconfig /renew
ipconfig /flushdns

# পাওয়ার কনফিগারেশন চেক করুন
powercfg -devicequery wake_armed
powercfg -devicedisablewake "Realtek PCIe GbE Family Controller"
powercfg -deviceenablewake "Intel(R) Dual Band Wireless-AC 8265"

# কম্পিউটার রিস্টার্ট করুন
shutdown /r /t 0
সমস্যা সমাধান চেকলিস্ট
BIOS-এ LAN/WLAN Auto Switching বন্ধ করা

HP Switching Service ডিজেবল করা (যদি HP হয়)

উভয় নেটওয়ার্ক অ্যাডাপ্টারের পাওয়ার ম্যানেজমেন্ট বন্ধ করা

নেটওয়ার্ক রিসেট করা

নেটওয়ার্ক ড্রাইভার আপডেট করা

অপ্রয়োজনীয় VPN সংযোগ মুছে ফেলা

প্রয়োজনীয় স্ক্রিপ্ট (একবারেই সব সমাধান)
powershell
# অ্যাডমিন PowerShell হিসেবে চালান

Write-Host "নেটওয়ার্ক সমস্যা সমাধান শুরু হচ্ছে..." -ForegroundColor Green

# 1. HP সার্ভিস বন্ধ করুন
Write-Host "HP Switching Service বন্ধ করা হচ্ছে..." -ForegroundColor Yellow
Get-Service "HP*switch*" -ErrorAction SilentlyContinue | Stop-Service
Get-Service "HP*switch*" -ErrorAction SilentlyContinue | Set-Service -StartupType Disabled

# 2. পাওয়ার ম্যানেজমেন্ট বন্ধ করুন
Write-Host "পাওয়ার ম্যানেজমেন্ট কনফিগার করা হচ্ছে..." -ForegroundColor Yellow
$adapters = Get-NetAdapter | Where-Object {$_.InterfaceDescription -like "*wireless*" -or $_.InterfaceDescription -like "*Ethernet*"}
foreach ($adapter in $adapters) {
    $adapterName = $adapter.Name
    Write-Host "কনফিগার করা হচ্ছে: $adapterName"
    Disable-NetAdapterPowerManagement -Name $adapterName -ErrorAction SilentlyContinue
}

# 3. নেটওয়ার্ক রিসেট
Write-Host "নেটওয়ার্ক রিসেট করা হচ্ছে..." -ForegroundColor Yellow
netsh int ip reset
netsh winsock reset
ipconfig /flushdns

Write-Host "সমাধান সম্পন্ন! কম্পিউটার রিস্টার্ট করুন।" -ForegroundColor Green
ফলো-আপ
যদি উপরের ধাপগুলো অনুসরণ করার পরও সমস্যা সমাধান না হয়, তাহলে নিচের তথ্যগুলো সংগ্রহ করে আমাদের জানান:

কম্পিউটারের মডেল ও ব্র্যান্ড

উইন্ডোজ ভার্সন (Win + R → winver)

নেটওয়ার্ক অ্যাডাপ্টারের মডেল

BIOS ভার্সন

দ্রষ্টব্য: এই সমাধান গাইডটি অনুসরণ করার সময় সতর্ক থাকুন এবং প্রয়োজনে বিশেষজ্ঞের সাহায্য নিন।

