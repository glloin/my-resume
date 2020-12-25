# 李龙

### QQ:254103979 13035206674

## 此项目主要在此分享我的项目经验

# 一元管理系统
* 卷材模块
* 板材生产模块
* 激光焊生产模块(一些材料需要拼焊后再进行冲压)
* 单品生产模块(就是把板材冲压成形状)
* Assy生产模块(就是把各种形状的产品拼焊在一起)
* 物流管理模块

## 卷材管理模块

## 项目介绍
* 主要用来应对当前社内购买卷材到卷材进入仓库的一整套流程。
### 先说一下当前业务流程
* 1. 计划科提供卷材计划
* 2. 购买科向各供应商购买卷材
* 3. 卷材供应商加工卷材
* 4. 卷材供应商发送到货清单，并送货
* 5. 冲压科进行卷材验收作业，对卷材进行称重，验收，并作书写记录。
* 6. 冲压科文员录入卷材验收作业员提交的记录，并与到货清单进行核对。将纸质票据归档。
* 7. 冲压使用卷材时对材料进行记录，每日作业完毕提交最终经过确认然后由文员录入数据归档。
  
### 当前存在的问题
* 1.冲压科作业员卷材上料作业时寻找卷材时间太长，因为很多卷材外观差异较小，寻找材料非常消耗时间。
* 2.对数据的采集录入，都是比较原始的方式，经常出现作业员书写错误导致月底盘点异常。
* 3.各科室不能够及时了解到当前卷材状态，比如原本应该今日到货的卷材未到，不能及时做出反应，很多时候模具装好了以后发现没有材料。或者本来仓库内有材料没有找到。
* 4.数据统计不灵活。当前的数据统计都是用Excel来完成。各个科室有各自的Excel文件。其实也是做了重复工作。

### 改善方案
* 服务： AspNetCore+SignalR
* 前端:  Webpack React TypeScript AntDesign Cass(不喜欢用less) 
* TV:    ReactNative
* PDA: Xamarin.Forms
* 可能大家会想为什么要用RN 又用Xamarin 也是我一开始比较纠结Xamarin的热加载 Xamarin的热加载只能加载Xaml。
* 做完了TV后感觉RN的热加载也很慢，其实用Xamarin也慢不了多少，反而由于C#特别熟练用Xamarin非常顺手，虽然React也很精通，但是仅限前端，感觉RN还是比较多的坑。
* PLC访问，也就是访问设备的plc 获取当前产品号和下一产品号根据不同的产品号让tv端指定的计划高亮置顶显示。

### PLC访问、硬件通讯相关
* 主要涉及三菱Q系列的PLC 
* 查阅了三零的 Q-系列MELSEC-通讯协议参考手册
* 其实通讯也挺简单的，Socket 按照协议 发送报文，解析返回的数据。
* 之后我专门报了一个 STM32的培训班，其实主要是想去深入了解通讯协议的。
* 项目里面还涉及了一个电子秤 宁波依托力的
* 直接联系了厂家，就把通讯协议给我了，设备是主动发送数据的。所以也非常简单。

### 数据模块
* 数据模型名称->数据库表名称
* 用户信息->dbo.UserInfo
* 车型->dbo.CarType 
* 部品信息->dbo.ProductInfo
* 卷材库位->Location.CoilLocation
* 卷材计划->Coil.Plan
* 到货清单->Coil.DeliveryInfo
* 卷材验收->Coil.CheckAcceptance
* 卷材库存->Coil.Store
* 卷材使用->Coil.UseOut
* 卷材退库->Coil.UseBack
* 卷材移库->Coil.MoveLocation

### 项目展示

* PDA 效果图
![pda](http://106.55.174.130/images/pda.png)

* Web 卷材数据管理页面
![coil display](http://106.55.174.130/images/coil_manager.png)
<a href="http://106.55.174.130/images/coil_manager.mp4">视频演示</a>

* 卷材现场电子看板视频
![coil manager](http://106.55.174.130/images/coilDisplay.png)
<a href="http://106.55.174.130/images/coilDisplay.mp4">视频演示</a>

* AspNetCore 注入冲压机PLC访问的后台服务
```
 public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
            //注入冲压PLC访问服务
            .ConfigureServices(s => s.AddPressHosts())
            ;
    }
```

* AspNetCore 注入相关服务
```
 public void ConfigureServices(IServiceCollection services)
        {
            var time = new Convert.DateTimeConverter();
            var timeNullable = new Convert.DateTimeNullableConverter();
            // 添加Api服务
            services.AddControllers().AddJsonOptions(config =>
            {
                config.JsonSerializerOptions.Converters.Add(time);
                config.JsonSerializerOptions.Converters.Add(timeNullable);
            });
            // 添加DbContext服务
            services.AddDbContext<UMDBContext>(optionAction =>
            {
                optionAction.UseSqlServer(_config.GetConnectionString("DBUnitaryManufacture"));
                optionAction.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
            });
            // 添加数据访问服务
            services.AddDalServices();
            // 添加即时通讯SignalR服务
            services.AddSignalR().AddJsonProtocol(builder =>
            {
                builder.PayloadSerializerOptions.Converters.Add(time);
                builder.PayloadSerializerOptions.Converters.Add(timeNullable);
            });
            // 添加 即时通讯 相关的调度服务，主要用于 api 方面向 hub推送消息，plc向 hub推送消息
            services.AddHubNotifys();
            // 添加权限验证服务
            services.AddMyAuth(_config);
        }
```
* 我自己搭的webpack 应对多页面开发
```

const htmlPlugin = (name, title) => new HtmlWebpackPlugin({
    chunks: [name],
    filename: `${name}.html`,
    template: resolve('./public/index.html'),
    inject: true,
    hash: true,
    favicon: resolve('./public/static/favicon.ico'),
    title
});

const pages = [
    { name: 'index', filename: resolve('./src/pages/main/index.tsx'), title: '东普雷(襄阳)一元管理系统' },
    { name: 'baseData', filename: resolve('./src/pages/baseData/index.tsx'), title: '基础数据' },
    { name: 'productInfo', filename: resolve('./src/pages/productInfo/index.tsx'), title: '部品信息管理' },
    { name: 'location', filename: resolve('./src/pages/location/index.tsx'), title: '库位管理' },
    { name: 'coil', filename: resolve('./src/pages/coil/main/index.tsx'), title: '卷材受入数据管理' },
    { name: 'coilCheck', filename: resolve('./src/pages/coil/check/index.tsx'), title: '卷材验收' },
    { name: 'coilUse', filename: resolve('./src/pages/coil/use/index.tsx'), title: '卷材使用' },
    { name: 'coilStoreDisplay', filename: resolve('./src/pages/coil/storeDisplay/index.tsx'), title: '卷材库存看板' },
    { name: 'pressProductionData', filename: resolve('./src/pages/press/productionData/index.tsx'), title: '冲压生产数据' },
    { name: 'display', filename: resolve('./src/pages/press/display/index.tsx'), title: '电子看板' },
    { name: 'pressPlanDisplay', filename: resolve('./src/pages/press/planDisplay/index.tsx'), title: '计划看板' },
    { name: 'monitor', filename: resolve('./src/pages/press/monitor/index.tsx'), title: '生产监视' },
    { name: 'pressPlan', filename: resolve('./src/pages/press/plan/index.tsx'), title: '冲压生产计划' },
    { name: 'user', filename: resolve('./src/pages/user/index.tsx'), title: '用户管理' },
];

const entry = {};
const htmlPlugins = [];

pages.forEach(a => {
    entry[a.name] = a.filename;
    htmlPlugins.push(htmlPlugin(a.name, a.title));
});

module.exports = {
    entry,
    output: {
        filename: 'static/js/[name]-topre.js',
        path: resolve('../Unitary.WebServer/wwwroot')
    },
    module: {
        rules: [
            {
                test: /\.(t|j)s(x?)$/,
                exclude: /node_modules/,
                loader: 'ts-loader',
                options: {
                    transpileOnly: true,
                    happyPackMode: true,
                    getCustomTransformers: () => ({
                        before: [tsImportPluginFactory({
                            libraryDirectory: 'es',
                            libraryName: 'antd',
                            style: true
                        })]
                    }),
                    compilerOptions: {
                        module: 'es2015'
                    }
                }
            },
            {
                test: /\.css/, use: [{
                    loader: MiniCssExtractPlugin.loader,
                    options: {
                        publicPath: '../'
                    },
                }, 'css-loader']
                , exclude: /node_modules/
            },
            {
                test: /\.less$/,
                include: resolve('./node_modules'),
                use: [{
                    loader: MiniCssExtractPlugin.loader,
                    options: {
                        publicPath: '../'
                    },
                }, 'css-loader', {
                    loader: 'less-loader',
                    options: {
                        lessOptions: {
                            modifyVars: {},
                            javascriptEnabled: true,
                        }
                    }
                }]
            },
            {
                test: /\.scss$/,
                include: resolve('./src'),
                use: [{
                    loader: MiniCssExtractPlugin.loader,
                    options: {
                        publicPath: '../'
                    },
                },
                    'css-loader',
                    'sass-loader'
                ]
            },
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            //小于10kb时打包成base64编码的图片否则单独打包成图片
                            limit: 10240,
                            name: 'static/img/[name].[hash:7].[ext]'
                        }
                    }]
            },
        ]
    },
    resolve: {
        extensions: ['.ts', '.tsx', '.js', '.jsx'],
        // alias: { "@": resolve("./src") }
        plugins: [new TsconfigPathsPlugin({ /*configFile: "./path/to/tsconfig.json" */ })]
    },
    plugins: [
        ...htmlPlugins,
        new Webpack.DllReferencePlugin({
            context: rootPath,
            manifest: resolve('./public/static/dll/vendor-manifest.json')
        }),
        new HtmlWebpackTagsPlugin({
            tags: ["./static/dll/vendor.dll.js"],
            append: false
        }),
        new CopyWebpackPlugin({
            patterns: [
                { from: resolve('./public/static/'), to: 'static/' },
            ]
        }),
        new ForkTsCheckerWebpackPlugin({
            typescript: {
                diagnosticOptions: {
                    semantic: true,
                    syntactic: true,
                },
            },
        }),
        new MiniCssExtractPlugin({
            filename: '[name].css',
            chunkFilename: '[id].css',
            ignoreOrder: false,
        })
    ],
    node: {
        fs: "empty"
    }
};
```

* RN项目 没什么可Show的，随便粘贴点RN代码算了
```
import React, { useEffect, useState } from 'react';
import { View, StyleSheet } from 'react-native';
import getBack from './components/back';
import Locations from './components/index';
import { IStandard } from './components/types';
import Hub, { methodsHandle } from './components/hub';
import { serverUrl } from '@/common/config';
export default () => {
    const [datas, setDatas] = useState<IStandard[]>([]);
    const [backimg, setBackimg] = useState<string>();
    useEffect(() => {
        const fn = () => {
            h.getBackImage().then((s: string) => setBackimg(`${serverUrl}${s}`)).catch((e: any) => console.log(e));
            h.getStandards().then(setDatas).catch((e: any) => console.log(e));
        };
        const h = Hub.getHub();
        const obj: methodsHandle = {
            onStart: fn,
            onreconnected: fn,
        };
        h.addHandle(obj);
        return () => {
            h.removeHandle(obj);
        };
    }, []);
    return (
        <View style={styles.root}>
            {backimg && getBack(backimg)}
            <Locations datas={datas} />
        </View>
    );
};
const styles = StyleSheet.create({
    text: {
        textAlign: 'center',
        textAlignVertical: 'center',
        color: '#f60',
        zIndex: 1000,
    },
    root: {
        flex: 1,
    },
});
```


