# CardiMind Multi-Agent医疗诊断系统

基于Multi-Agent架构和RAG（检索增强生成）技术的智能医疗诊断系统，能够模拟多位医生的协作诊断过程，提供高质量的医疗诊断分析。

## 🚀 系统特性

### Multi-Agent协作诊断
- **专业分工**：4个专业Agent模拟真实临床诊断流程
- **流程化诊断**：首诊发散 → 循证检索 → 质疑挑战 → 综合决策
- **质量控制**：通过质疑和挑战机制降低误诊风险
- **证据驱动**：基于证据强度进行诊断决策

### RAG技术支持
- **PDF文档处理**: 自动提取和处理corpus文件夹中的PDF医疗文献
- **智能向量化**: 使用gme-qwen2-vl-7b模型进行文本嵌入
- **高效检索**: 基于余弦相似度的向量检索系统
- **智能问答**: 使用deepseek-v3-0324模型生成专业回答

### 多种使用方式
- **Web界面**: 基于Streamlit的友好用户界面
- **API服务**: RESTful API支持系统集成
- **命令行工具**: 支持单文件和批量测试
- **Docker部署**: 完整的容器化解决方案

## 📋 系统要求

- Python 3.8+
- 网络连接（用于API调用）
- 足够的磁盘空间存储向量数据库
- 推荐至少4GB内存和稳定的网络连接

## 🛠️ 安装和使用

### 1. 快速启动

最简单的启动方式：

```bash
python start.py
```

该脚本会自动：
- 检查系统要求
- 安装依赖包
- 创建必要目录
- 测试系统组件
- 构建知识库
- 启动Web界面

### 2. 手动安装

如果需要手动安装：

```bash
# 安装依赖
pip install -r requirements.txt

# 启动Web界面
streamlit run main_app.py
```

### 3. 启动Multi-Agent API服务

```bash
# 启动服务器（默认端口8080）
python start_cardiomind.py

# 指定端口和主机
python start_cardiomind.py --host 0.0.0.0 --port 8080

# 启用调试模式
python start_cardiomind.py --debug

# 运行系统测试
python start_cardiomind.py --test
```

### 4. Docker容器化部署

#### 方法一：使用预构建的Docker镜像（推荐）

```bash
# 加载预构建的Docker镜像
docker load -i qyj-Cardiomind-Agent.tar

# 查看已加载的镜像
docker images | grep cardiomind

# 启动完整系统（API + Web界面）
docker run -d \
  --name cardiomind-system \
  -p 8080:8080 \
  -p 8501:8501 \
  -v $(pwd)/logs:/app/logs \
  -v $(pwd)/medical_output:/app/medical_output \
  -v $(pwd)/vector_db:/app/vector_db \
  -e STARTUP_MODE=full \
  qyj-cardiomind-agent:latest

# 仅启动API服务
docker run -d \
  --name cardiomind-api \
  -p 8080:8080 \
  -v $(pwd)/logs:/app/logs \
  -v $(pwd)/medical_output:/app/medical_output \
  -e STARTUP_MODE=api \
  qyj-cardiomind-agent:latest

# 仅启动Web界面
docker run -d \
  --name cardiomind-web \
  -p 8501:8501 \
  -v $(pwd)/logs:/app/logs \
  -e STARTUP_MODE=streamlit \
  qyj-cardiomind-agent:latest

# 查看容器运行状态
docker ps

# 查看容器日志
docker logs cardiomind-system

# 健康检查
curl -X GET "http://localhost:8080/health"
```

#### 方法二：从源码构建镜像

```bash
# 构建镜像
docker build -t medical-diagnosis-system .

# 使用 Docker Compose（推荐）
docker-compose up -d

# 直接运行容器
docker run -d -p 8080:8080 -p 8501:8501 --name medical-system medical-diagnosis-system
```

#### Docker部署配置选项

**环境变量**：
- `STARTUP_MODE`: 启动模式
  - `full`: 启动API服务和Web界面（默认）
  - `api`: 仅启动API服务
  - `streamlit`: 仅启动Web界面

**数据持久化**：
```bash
# 创建数据目录
mkdir -p logs medical_output vector_db

# 启动时挂载数据卷
docker run -d \
  --name cardiomind-system \
  -p 8080:8080 -p 8501:8501 \
  -v $(pwd)/logs:/app/logs \
  -v $(pwd)/medical_output:/app/medical_output \
  -v $(pwd)/vector_db:/app/vector_db \
  -v $(pwd)/medical_records:/app/medical_records \
  qyj-cardiomind-agent:latest
```

#### 快速测试部署

```bash
# 1. 加载镜像
docker load -i qyj-Cardiomind-Agent.tar

# 2. 启动系统
docker run -d --name cardiomind -p 8080:8080 -p 8501:8501 qyj-cardiomind-agent:latest

# 3. 等待系统启动（约30-60秒）
sleep 60

# 4. 健康检查
curl -X GET "http://localhost:8080/health"

# 5. 测试诊断功能（需要准备测试病历文件）
curl -X POST "http://localhost:8080/cardiomind" \
     -H "Content-Type: application/json" \
     -d '{"medical record": "患者男性，65岁，主诉胸闷气喘3天..."}'

# 6. 访问Web界面
echo "Web界面地址: http://localhost:8501"
```

## 🏥 Multi-Agent系统架构

系统包含4个专业Agent，模拟真实的临床诊断流程：

### 1. Dr.Hypothesis（首诊发散者）
- **角色**：根据患者主诉与病史快速列出Top-N诊断假设
- **功能**：不负责求证，专注于发散思维，生成尽可能多的诊断可能性
- **优化**：增强了JSON解析稳定性和关键信息提取准确性

### 2. Dr.Evidence（循证图书管理员）
- **角色**：针对每个假设自动调用RAG系统召回指南条件
- **功能**：唯一与RAG耦合的节点，检索诊断标准、排除标准与推荐检查
- **优化**：改进了证据分析的稳定性和结构化输出

### 3. Dr.Challenger（挑刺副高）⭐ **新优化**
- **角色**：对照指南原文检查假设是否成立
- **功能**：提出反证与替代诊断，降低误诊风险
- **新增功能**：
  - **证据强度质疑**：评估主诊断的证据权重（临床症状优先40%）
  - **遗漏诊断提醒**：基于轻量级知识库自动检测被忽视的明显诊断
  - **逻辑一致性验证**：检查病理生理逻辑、时间逻辑、严重程度逻辑

### 4. Dr.ClinicalReasoner（主任医师）⭐ **新优化**
- **角色**：综合假设、证据、挑战与结构化数据
- **功能**：输出符合标准格式的主/次要诊断、鉴别诊断及置信度
- **新增功能**：
  - **证据驱动诊断**：基于证据强度重新排序诊断优先级
  - **严格格式合规**：确保输出100%符合schema，自动清理所有格式干扰
  - **字段交叉验证**：验证体格检查、辅助检查等字段的逻辑一致性

## 🌐 访问地址

- **Flask API服务**: http://localhost:8080
- **Streamlit Web应用**: http://localhost:8501
- **健康检查**: http://localhost:8080/health

## 📖 API接口详细说明

### 核心诊断服务

#### POST /cardiomind
- **功能**: 执行完整的Multi-Agent医疗诊断流程
- **输入**: JSON格式的医疗记录
- **输出**: 符合标准格式的综合诊断结果

**输入格式**:
```json
{
  "medical record": "病历内容字符串"
}
```

**输出格式**:
```json
{
  "患者信息": {
    "年龄": 52,
    "性别": "男",
    "入院日期": "2022-04-01"
  },
  "临床表现": {
    "主诉": "反复腹痛、腹胀2周余",
    "现病史": "患者2周余前无明显诱因反复出现腹痛、腹胀..."
  },
  "病史信息": {
    "既往史": "高血压3年，血压最高可达160/100mmHg...",
    "个人史": "无特殊，否认吸烟、饮酒史。",
    "婚育史": "适龄结婚，育有1子。",
    "家族史": "父亲因心肌梗死去世，母亲健在..."
  },
  "体格检查": "体温37.0℃，心率96次/分，呼吸20次/分...",
  "辅助检查": "急诊化验检查：总胆红素51.92μmol/L...",
  "诊断结果": {
    "主要诊断": {
      "名称": "失代偿期肝硬化",
      "诊断依据": [
        "既往有丙型肝炎、布加综合征病史，均为肝硬化高危因素",
        "腹部CT提示肝硬化、脾大、腹水、侧支循环形成"
      ]
    },
    "次要诊断": [...],
    "鉴别诊断": [...]
  },
  "治疗方案": [...],
  "_system_metadata": {
    "workflow_id": "2025-09-14T09:04:55.123456",
    "agents_executed": ["Dr.Hypothesis", "Dr.Evidence", "Dr.Challenger", "Dr.ClinicalReasoner"],
    "api_version": "1.0.0",
    "processing_time_seconds": 45.678
  }
}
```

### 辅助服务端点

#### GET /health
- **功能**: 系统健康检查
- **输出**: 系统状态信息

#### GET /status  
- **功能**: 获取详细系统状态
- **输出**: 各Agent状态和系统配置信息

#### POST /test
- **功能**: 运行系统测试
- **输入**: 可选的测试用例JSON
- **输出**: 测试结果报告

## 🧪 测试方法

### 1. 系统综合测试

```bash
# 运行完整系统测试
python test_multi_agent.py

# 测试优化功能
python test_challenger_optimized.py
python test_clinical_reasoner_optimized.py
```

### 2. 单个病历测试

使用 `single_medical_test.py` 测试指定的病历文件：

```bash
# 基本用法
python single_medical_test.py case1.json

# 测试其他病例
python single_medical_test.py case2.json
python single_medical_test.py case3.json

# 确保API服务已启动
python start_cardiomind.py &
sleep 5
python single_medical_test.py case1.json
```

**功能特点**：
- 📖 自动读取 `medical_records/` 目录下的病历文件
- 🔍 检查API服务健康状态
- 🩺 调用诊断API进行分析
- 💾 自动保存结果到 `medical_output/` 目录
- ⚠️ 错误处理和日志记录

**输出文件格式**：
```
medical_output/
├── case1_result_20250914_123456.json    # 成功结果
├── case1_error_20250914_123456.json     # 错误日志
```

### 3. API调用测试

#### 使用现有测试病例
```bash
# 方法一：使用case1.json（推荐）
curl -X POST "http://localhost:8080/cardiomind" \
     -H "Content-Type: application/json" \
     --data-binary "@medical_records/case1.json"

# 方法二：使用其他测试病例
curl -X POST "http://localhost:8080/cardiomind" \
     -H "Content-Type: application/json" \
     --data-binary "@medical_records/case2.json"
```

#### 使用自定义病历
```bash
# 创建自定义病历文件
cat > my_case.json << 'EOF'
{
  "medical record": "患者男性，65岁，主诉胸闷气喘3天。现病史：患者3天前出现胸闷气喘，活动后加重，伴有夜间阵发性呼吸困难，无胸痛，无发热。既往有高血压病史10年，糖尿病史5年。体格检查：血压160/95mmHg，心率110次/分，双肺底可闻及湿性啰音，心界向左扩大，可闻及第三心音奔马律。辅助检查：心电图提示左心室肥厚，胸片示心影扩大，肺淤血征象。"
}
EOF

# 调用诊断服务
curl -X POST "http://localhost:8080/cardiomind" \
     -H "Content-Type: application/json" \
     --data-binary "@my_case.json"
```

#### 批量处理
```bash
# 处理多个病例
for case in medical_records/case*.json; do
    echo "处理病例: $case"
    python single_medical_test.py $(basename "$case")
    echo "---"
done
```

### 4. Windows/Linux脚本

**Windows用户**：
```cmd
# 运行完整API测试
test_api.bat

# 手动执行curl命令
curl -X POST "http://localhost:8080/cardiomind" ^
     -H "Content-Type: application/json" ^
     --data-binary "@medical_records/case1.json"
```

**Linux/Mac用户**：
```bash
# 使用提供的shell脚本
chmod +x test_api.sh
./test_api.sh

# 直接执行curl命令
curl -X POST "http://localhost:8080/cardiomind" \
     -H "Content-Type: application/json" \
     --data-binary "@medical_records/case1.json"
```

## 📁 项目结构

```
医疗诊断系统/
├── agents/                     # Agent模块
│   ├── __init__.py
│   ├── base_agent.py          # 基础Agent类
│   ├── dr_hypothesis.py       # 首诊发散者
│   ├── dr_evidence.py         # 循证图书管理员  
│   ├── dr_challenger.py       # 挑刺副高（已优化）
│   └── dr_clinical_reasoner.py # 主任医师（已优化）
├── multi_agent_system.py      # Multi-Agent协调系统
├── cardiomind_api.py          # Flask API服务
├── start_cardiomind.py        # API服务启动脚本
├── corpus/                    # PDF医疗文献存放目录
│   ├── *.pdf                  # 医疗文献PDF文件
├── vector_db/                 # 向量数据库存储目录
├── medical_records/           # 测试病例库
│   ├── case1.json            # 主要测试病例
│   ├── case2.json            # 其他测试病例
│   └── ...
├── medical_output/            # 诊断结果输出目录
├── logs/                      # 系统日志目录
├── pdf_extractor.py          # PDF文本提取器
├── embedding_service.py      # 嵌入服务
├── vector_database.py        # 向量数据库
├── llm_service.py            # LLM服务
├── rag_pipeline.py           # RAG管道
├── main_app.py               # Streamlit主应用程序
├── config.py                 # 配置文件
├── start.py                  # 系统启动脚本
├── single_medical_test.py    # 单文件测试工具
├── test_multi_agent.py       # 系统测试脚本
├── test_api.sh/.bat          # API测试脚本
├── test_challenger_optimized.py    # Dr.Challenger优化测试
├── test_clinical_reasoner_optimized.py # Dr.ClinicalReasoner优化测试
├── Dockerfile                # Docker容器配置
├── docker-compose.yml        # Docker Compose配置
├── requirements.txt          # 依赖包列表
└── README.md                 # 说明文档
```

## 💡 使用指南

### 1. 添加文献
将PDF医疗文献文件放入`corpus/`文件夹中。

### 2. 构建知识库
首次使用或添加新文献后，系统会自动构建知识库。也可以在Web界面中手动重建。

### 3. 开始问答
在Web界面的聊天框中输入医疗相关问题，例如：
- "什么是主动脉瓣置换术？"
- "冠状动脉旁路移植术后的康复要点是什么？"
- "心脏康复的基本原则是什么？"

### 4. 查看参考文献
系统会显示回答所基于的具体文献来源和相似度得分。

### 5. API集成使用

#### Python调用示例
```python
from multi_agent_system import create_diagnosis_system

# 创建系统
system = create_diagnosis_system()

# 准备病历
medical_record = {
    "medical record": "患者男，65岁，主诉胸闷气喘3天，现病史：患者3天前出现胸闷气喘..."
}

# 执行诊断
result = system.diagnose(medical_record)
print("主要诊断:", result["诊断结果"]["主要诊断"]["名称"])
print("诊断依据:", result["诊断结果"]["主要诊断"]["诊断依据"])
```

## ⚙️ 配置说明

在`config.py`中可以调整以下参数：

```python
# 文本分块设置
CHUNK_SIZE = 1000           # 文本块大小
CHUNK_OVERLAP = 200         # 文本块重叠

# 检索设置
DEFAULT_TOP_K = 5           # 检索文档数量
DEFAULT_MIN_SIMILARITY = 0.1 # 最小相似度阈值

# LLM服务配置
LLM_API_BASE_URL = "https://api.juheai.top"
LLM_MODEL = "deepseek-v3-0324"

# 嵌入服务配置
EMBEDDING_API_BASE_URL = "https://gme-qwen2-vl-7b.ai4s.com.cn"
```

## 🔍 高级功能

### 系统状态检查
Web界面侧边栏提供：
- 各组件状态检查
- 知识库信息统计
- 性能监控

### 知识库管理
- 查看文档统计信息
- 重建知识库
- 数据库状态监控

### 对话管理
- 多轮对话支持
- 对话历史记录
- 清空对话功能

### 最新优化功能

#### Dr.Challenger增强
- **证据强度质疑**：多维度评估诊断证据（临床症状40%、体格检查30%、实验室20%、影像10%）
- **遗漏诊断检测**：基于症状-疾病映射自动发现被忽视的诊断
- **逻辑一致性验证**：检查病理生理逻辑、时间逻辑、严重程度排序

#### Dr.ClinicalReasoner增强  
- **证据权重整合**：综合初始概率(40%)、证据强度(40%)、质疑分析(20%)
- **格式严格合规**：100%符合output_example.json格式，自动清理markdown语法
- **智能字段验证**：确保体格检查、辅助检查等字段内容准确分离

## 📊 性能指标

### 系统性能
- **平均处理时间**：30-60秒（取决于病历复杂度）
- **并发处理能力**：支持多线程并发请求
- **内存占用**：约500MB-1GB
- **准确性**：基于优化后的证据驱动决策机制

### API响应时间
- **健康检查**：< 100ms
- **系统状态查询**：< 500ms  
- **诊断请求**：30-60秒
- **测试接口**：5-10秒

## ⚠️ 注意事项

1. **免责声明**：本系统仅供学术研究和参考使用，不能替代专业医疗建议
2. **数据隐私**：请确保测试数据已去除个人隐私信息
3. **模型限制**：诊断结果的准确性依赖于训练数据和模型能力
4. **网络要求**：系统需要访问LLM API，请确保网络连接正常
5. **资源要求**：建议至少4GB内存和稳定的网络连接
6. **API限制**: 注意API调用频率限制，系统已内置延迟和重试机制
7. **存储空间**: 向量数据库可能占用较大磁盘空间

## 🐛 故障排除

### 常见问题

1. **Docker镜像部署问题**
   ```bash
   # 镜像加载失败
   docker load -i qyj-Cardiomind-Agent.tar
   # 如果报错，检查文件是否完整
   
   # 容器启动失败
   docker logs cardiomind-system
   # 查看详细错误日志
   
   # 端口占用
   docker run -d --name cardiomind -p 8081:8080 -p 8502:8501 qyj-cardiomind-agent:latest
   # 使用不同的端口映射
   
   # 清理容器和镜像
   docker stop cardiomind-system
   docker rm cardiomind-system
   docker rmi qyj-cardiomind-agent:latest
   ```

2. **服务启动失败**
   ```bash
   # 检查端口占用
   netstat -an | grep :8080
   # 或使用其他端口
   python start_cardiomind.py --port 8081
   ```

2. **API调用失败**
   - 检查网络连接
   - 验证API密钥
   - 查看日志信息

3. **PDF提取失败**
   - 确保PDF文件未损坏
   - 检查文件权限
   - 查看错误日志

4. **知识库构建失败**
   - 检查corpus文件夹是否存在
   - 确保有足够的磁盘空间
   - 重试构建过程

5. **LLM连接错误**
   ```bash
   # 运行连接测试
   python test_llm_quick.py
   # 检查config.py中的API配置
   ```

6. **诊断质量问题**
   ```bash
   # 运行优化测试验证功能
   python test_challenger_optimized.py
   python test_clinical_reasoner_optimized.py
   ```

7. **单文件测试错误**
   ```bash
   # 常见错误及解决方案
   
   # 连接错误
   curl: (7) Failed to connect to localhost port 8080
   # 解决：确保服务已启动并监听8080端口
   
   # 文件不存在
   ❌ 文件不存在: medical_records/case1.json
   # 解决：确保在项目根目录执行，或使用绝对路径
   
   # JSON格式错误
   {"error": "JSON格式错误: ..."}
   # 解决：检查JSON文件格式，确保符合要求
   ```

### 日志查看

```bash
# 查看API日志
tail -f logs/api.log

# 查看Agent运行日志  
tail -f logs/agents.log

# 查看系统日志输出
tail -f logs/debug.log
```

## 📞 技术支持

如果遇到问题，请检查：
1. 系统日志输出
2. 网络连接状态
3. API服务可用性
4. 文件权限设置
5. 健康检查：`GET /health` 端点检查系统状态
6. 测试功能：使用 `--test` 参数进行系统诊断
7. 优化验证：运行专门的优化测试脚本

## 🔄 更新日志

- **v1.2.0**: 重大优化更新
  - Dr.Challenger增强：证据强度质疑、遗漏诊断检测、逻辑一致性验证
  - Dr.ClinicalReasoner优化：证据驱动诊断、严格格式合规、字段交叉验证
  - API稳定性改进和错误处理优化
  - 添加Docker容器化支持
  - 增加单文件测试工具

- **v1.1.0**: 稳定性提升  
  - 改进JSON解析稳定性
  - 增强错误处理机制
  - 优化API响应格式

- **v1.0.0**: 初始版本发布
  - 完整的RAG系统实现
  - Web界面支持
  - 多种医疗文献格式支持
  - 实现Multi-Agent诊断系统
  - 支持4个专业Agent的协作诊断
  - 集成RAG检索系统
  - 提供标准化API接口