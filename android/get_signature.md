# 获取程序相关信息(kotlin实现)

``` java

## 获取byteArray的md5值
private fun md5Encrypt(byteArray:ByteArray):String{
    val md5Digest = MessageDigest.getInstance("MD5")
    md5Digest.reset()
    val digestedBytes = md5Digest.digest(byteArray)
    val StringBuffer = StringBuffer()
    for ( i in 0 until digestedBytes.size){
        val s = Integer.toHexString(0xFF and digestedBytes[i].toInt)
        if(s.length == 1){
            stringBuffer.append("0").append(s)
        }else{
            stringBuffer.append(s)
        }
    }
    return stringBuffer.toString()
}

## 获取所有安装包信息
val installedPackages:List<PackageInfo> = Context.packageManager
.getInstalledPackages(PackageManager.GET_SIGNATURE)

##过滤系统预装应用
val withoutSystemPackages:List<PackageInfo> = installedPackages.filter{
    it.applicationInfo.flags and ApplicationInfo.FLAG_SYSTEM == 0
}

## 根据PackageInfo获取签名
withoutSystemPacksges.map{
    val signatureMd5 = md5Encrypt(it.signatures[0].tyByteArray())
    val packageMd5 = md5Encrypt(File(it.appcationInfo.sourceDir).readBytes())
    val packageName = it.packageName
    val appName = it.applicationInfo.loadLabel(packageManager)
    val version = it.version?: "0.0.0"
    {
        "signatureMd5" to signatureMd5,
        "packageMd5" to packageMd5,
        "packageName" to packageName,
        "appName" to appName,
        "version" to version
    }
}

```