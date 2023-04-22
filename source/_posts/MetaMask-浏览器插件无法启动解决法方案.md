---
title: MetaMask 浏览器插件无法启动解决法方案
date: 2023-04-22 14:34:25
tags: 
- block chain
- 区块链
- 钱包
categories:
- 区块链
---

在没有备份过助记词/私钥，同时重启插件/电脑均无法解决的情况下，可在电脑本地全局搜索 nkbihfbeogaeaoehlefnkodbefgpgknn，这是 MM 扩展 id，如这个目录下：C:\Users\[User]\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nkbihfbeogaeaoehlefnkodbefgpgknn 找到 ldb/log 这些文件，在这些文件里找到如图目标内容。 后用 https://metamask.github.io/vault-decryptor/解开这段目标内容，Password 就是目标 MetaMask 扩展的密码。 

```
{"data":"6vllhjdFt2RX/g6yxFN9CqSpKJlzzuXaRqeClxpEhZtPv3zLVib4Eq/sOpfcERJu01iuv37WT8w+kponTH+5IMH8DWSUr3FsmxcjQCJTJoc7E/iBx9fki1o9KaYtrWZcsMf3X1fGurR9lUIqtAKNZLOqDKY4POAszB+KerAGKVnzpJ6FhcahTgHzYkQeJ9Q+qzoO0VHxAqbeS1ZJhgmBtln92XV/MUXEPF1AcbJF76m10JQVo+sg1u4iOapxNep+R/K+HopMpoyB8Q6HluV23wyxbCXA6kEEuwJ91LLpRi7EFLoNhJK1/lTG5+CrXFZE/WYQ6+gFlzSlwqIKPr2cnbUwktsj2/rFJ5sgmnVRMzdx+JseQO8e8YZhveABkzexQ2kKr+auWYKZYThACPCPy8Z7yBXPL3jd6AVK1Ubc/xVVGY1n73dwIkOY6PO+/OcS++XQ/QwvCCDVSTshHxtumSqVNfxrTeI8RKwV3xPyh0BIuXY6xn+5Di58MLpREesEwd4nOXKn6Xw5SqNuEsUS7Sjh8RhD01krg0GSCQVnfWFhMTQH/Q/euCHo9OA5yuwG6iyBBAl9WwLcaypZuOibPc+MDkcHp9V+rwP0SSEF8dF2gQgP//jaPYZm6bm70SWwndzHE8g0HZhiy3kCnm1LP0YMpH5vLoD/Fb6OxXsKsOdUR9Jzl/xIJKJA1JqSkHHnDPELceaYJFX6vkbevstBg6CuY+FlfBqG2UtMCM19bLpaBQ134lZkAUUdHPsWRjgSWHWvqco8A1E9RjdMrMMzQrIMDaaP82UPNkXaU19E2Ij+I5jvr26PNMSL9qnGdmC6+kxkr/MBUpWtK2DUzU2ZGnVImqms3De0Is9ldLnbQmp8bOYwq+XT5aOYu7aSR10y5mKH6B2V4swBKqZIuGFHpGkGkXYQsoY1wQ+BNQbMX7pPXO5Wiq4N7smYYZ3a3PvmX6KelE3f8+1ulVg1r+1s0PmnFxFr+6tog2IDPdrJL8lfqpTtcDRXIaMsp+Tj24+y9OIZFktsTkqdAzaMfbtg==","iv":"kleBujFi5T+123iZU5CVEw==","salt":"WVsdQtIhPkDiJfN5ORg9Eubm5jqyKFm3J9oWNLRKc="}
```



若 MM 的任意扩展页面可以打开，比如：chrome-extension://nkbihfbeogaeaoehlefnkodbefgpgknn/home.html，则用下面的方式也有得到待解密的内容： 

```
chrome.storage.local.get('data', result => {  
    var vault = result.data.KeyringController.vault  
    console.log(vault) })
```


