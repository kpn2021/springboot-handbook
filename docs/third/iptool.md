# 根据IP定位地理位置
## 使用ip2region查询IP对应的地理位置

1.首先项目引入maven依赖
```
        <dependency>
            <groupId>org.lionsoul</groupId>
            <artifactId>ip2region</artifactId>
            <version>1.7.2</version>
        </dependency>
```
2.下载IP库文件(ip2region.db) 
[下载地址]( https://gitee.com/lionsoul/ip2region/tree/master/data) 

3.把 ip2region.db 复制到 maven 项目的 resources 目录下
![ip2region.db](https://s3.bmp.ovh/imgs/2022/01/123cda6ff668c142.png)

4.代码使用ip2region
```
import java.io.File;
import java.lang.reflect.Method;
import org.lionsoul.ip2region.DataBlock;
import org.lionsoul.ip2region.DbConfig;
import org.lionsoul.ip2region.DbSearcher;
import org.lionsoul.ip2region.Util;
/**
 * @author wy126
 */
public class Ip2RegionUtil {
    public static String getCityInfo(String ip) {
        //db
        String dbPath = Ip2RegionUtil.class.getResource("/ip2region.db").getPath();
        File file = new File(dbPath);
        if (!file.exists()) {
            System.out.println("Error: Invalid ip2region.db file");
        }
        try {
            DbConfig config = new DbConfig();
            DbSearcher searcher = new DbSearcher(config, dbPath);
            Method method = null;
            method = searcher.getClass().getMethod("btreeSearch", String.class);
            DataBlock dataBlock = null;
            if (!Util.isIpAddress(ip)) {
                System.out.println("Error: Invalid ip address");
            }
            dataBlock = (DataBlock) method.invoke(searcher, ip);
            return dataBlock.getRegion();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

## 调用第三方API获取IP对应的地理位置
> [太平洋网络IP地址查询](http://whois.pconline.com.cn/)

> [淘宝IP地址库](https://ip.taobao.com/)