# 接口自动化测试项目说明

## 📋 项目概述
本Postman集合实现了天气查询WebService接口的自动化测试链，包含`getRegionProvince` -> `getSupportCityString` -> `getWeather`的接口关联测试。结合Newman和Jenkins可实现持续集成测试。

---

## 🛠 技术栈
- **Postman** - 接口测试用例开发
- **Newman** - 命令行测试运行器
- **Jenkins** - 持续集成平台
- **XML处理** - 接口响应数据解析

---

## 📂 目录结构
```
├── weather_api_collection.json  # Postman集合文件
├── README.md                    # 项目说明文档
└── env.json                     # 环境变量文件（可选）
```

---

## 🔧 环境准备

### 1. 安装Node.js
```bash
# 从Node.js官网下载安装包
https://nodejs.org/
```

### 2. 安装Newman
```bash
npm install -g newman
```

### 3. 验证安装
```bash
newman --version
```

---

## 🚀 运行测试

### 基础运行命令
```bash
newman run weather_api_collection.json
```

### 带环境变量运行（推荐）
```bash
newman run weather_api_collection.json -e env.json
```

### 生成HTML报告
```bash
newman run weather_api_collection.json -r html,cli
```

---

## 🔄 Jenkins集成配置

### 1. 安装插件
- NodeJS Plugin
- HTML Publisher Plugin

### 2. 配置全局工具
`Manage Jenkins` > `Global Tool Configuration` 添加Node.js安装路径

### 3. 创建Pipeline任务
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://your-repo-url.git'
            }
        }
        
        stage('Test') {
            steps {
                script {
                    sh 'newman run weather_api_collection.json -r html,cli'
                }
            }
        }
        
        stage('Publish Report') {
            steps {
                publishHTML(
                    target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'newman',
                        reportFiles: 'report.html',
                        reportName: 'API Test Report'
                    ]
                )
            }
        }
    }
}
```

---

## 🔗 接口说明

### 1. getRegionProvince（获取省份列表）
- **Method**: GET
- **Endpoint**: `{{weater}}/WeatherWS.asmx/getRegionProvince`
- **断言**:
  - 响应状态码200
  - 响应包含"广东"
  - 响应时间<200ms

### 2. getSupportCityString（获取城市列表）
- **Method**: POST
- **Endpoint**: `{{weater}}/WeatherWS.asmx/getSupportCityString`
- **参数**: theRegionCode=31124
- **数据提取**:
  ```javascript
  var jsonObject = xml2Json(responseBody);
  var code = jsonObject.ArrayOfString.string[0].split(',')[1];
  pm.environment.set('CityCode', code); // 设置CityCode环境变量
  ```

### 3. getWeather（获取天气信息）
- **Method**: POST
- **Endpoint**: `{{weater}}/WeatherWS.asmx/getWeather`
- **参数**: 
  - theCityCode={{CityCode}}
  - theUserID=空值

---

## ⚠️ 注意事项
1. 确保测试服务器`http://ws.webxml.com.cn`可访问
2. 环境变量`weater`已正确配置
3. XML解析依赖Postman内置的`xml2Json`函数
4. 建议每日定时执行（通过Jenkins Cron配置）

---

## 📊 测试报告示例
![Newman HTML Report](https://user-images.githubusercontent.com/25589559/123543937-07a3a100-d789-11eb-8f4f-3a9a0a8d8e7a.png)

---

## 📬 问题反馈
如果有问题可以直接issue留言

---

通过此文档，您可以快速部署基于Newman+Jenkins的接口自动化测试流水线，实现接口测试的持续集成。建议结合实际项目需求调整测试数据和断言条件。
