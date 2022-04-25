### navicat的无限重置

针对最新的 navicat 16 的激活。可以使用无线重置脚本。在 15 之前有注册机激活。不过建议使用最新的。

```shell
@echo off
echo Delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration[version and language]
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium" /s | findstr /L Registration"') do (
    reg delete %%i /va /f
)
echo Delete Info folder under HKEY_CURRENT_USER\Software\Classes\CLSID
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\Classes\CLSID" /s | findstr /E Info"') do (
    reg delete %%i /va /f
)
echo Finish
pause
```

在 win 的环境之下，创建一个 txt 的文本，然后粘贴上面的内容，然后把 txt 的后缀改成 bat 。双击执行就可以了。

### idea的在线激活

```html
https://license-server.tmk.edu.hk
```

在 License server 选项中填入上面的连接。要是出现错误，多试几次就可以了。

