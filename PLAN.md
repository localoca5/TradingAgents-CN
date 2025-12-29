                                                                 │
     │                                                                                                   │
     │ 1.2 修改TradingAgentsGraph初始化                                                                  │
     │                                                                                                   │
     │ 文件: D:\projects\TradingAgents-CN-main\tradingagents\graph\trading_graph.py                      │
     │ def __init__(                                                                                     │
     │     self,                                                                                         │
     │     selected_analysts=["market", "social", "news", "fundamentals"],                               │
     │     debug=False,                                                                                  │
     │     config: Dict[str, Any] = None,                                                                │
     │     language: str = "zh-CN"  # 新增：语言参数                                                     │
     │ ):                                                                                                │
     │     self.debug = debug                                                                            │
     │     self.config = config or DEFAULT_CONFIG                                                        │
     │     # 将language添加到config中                                                                    │
     │     self.config["language"] = language                                                            │
     │                                                                                                   │
     │ 1.3 修改GraphSetup传递语言参数                                                                    │
     │                                                                                                   │
     │ 文件: D:\projects\TradingAgents-CN-main\tradingagents\graph\setup.py                              │
     │ # 在创建Agent节点时，传递language参数                                                             │
     │ language = config.get("language", "zh-CN")                                                        │
     │ market_analyst = create_market_analyst(llm, toolkit, language=language)                           │
     │                                                                                                   │
     │ 阶段2：修改核心Agent（4-6小时）                                                                   │
     │                                                                                                   │
     │ 2.1 修改市场分析师 (MVP优先级：高)                                                                │
     │                                                                                                   │
     │ 文件: D:\projects\TradingAgents-CN-main\tradingagents\agents\analysts\market_analyst.py           │
     │                                                                                                   │
     │ 修改内容：                                                                                        │
     │ 1. 修改create_market_analyst()函数签名，添加language参数                                          │
     │ 2. 修改prompt模板，将硬编码的"请使用中文"改为动态语言指令                                         │
     │ 3. 添加投资建议映射表                                                                             │
     │                                                                                                   │
     │ 代码示例：                                                                                        │
     │ def create_market_analyst(llm, toolkit, language="zh-CN"):                                        │
     │     # 投资建议映射                                                                                │
     │     investment_map = {                                                                            │
     │         "zh-CN": {"buy": "买入", "hold": "持有", "sell": "卖出"},                                 │
     │         "en-US": {"buy": "Buy", "hold": "Hold", "sell": "Sell"}                                   │
     │     }                                                                                             │
     │                                                                                                   │
     │     # 语言指令                                                                                    │
     │     language_instruction = "请使用中文，基于真实数据进行分析。" if language == "zh-CN" else       │
     │ "Please use English for analysis based on real data."                                             │
     │                                                                                                   │
     │     # 在prompt模板中使用 {language_instruction} 变量                                              │
     │     prompt = ChatPromptTemplate.from_messages([                                                   │
     │         ("system", f"...{language_instruction}"),                                                 │
     │     ])                                                                                            │
     │                                                                                                   │
     │     # 设置partial变量                                                                             │
     │     prompt = prompt.partial(language=language)                                                    │
     │                                                                                                   │
     │ 2.2 修改基本面分析师 (MVP优先级：高)                                                              │
     │                                                                                                   │
     │ 文件: D:\projects\TradingAgents-CN-main\tradingagents\agents\analysts\fundamentals_analyst.py     │
     │                                                                                                   │
     │ 修改内容：同上，修改prompt中的语言要求部分                                                        │
     │                                                                                                   │
     │ 2.3 修改信号处理模块                                                                              │
     │                                                                                                   │
     │ 文件: D:\projects\TradingAgents-CN-main\tradingagents\graph\signal_processing.py                  │
     │                                                                                                   │
     │ 修改内容：添加多语言投资建议映射                                                                  │
     │ def get_action_map(language="zh-CN"):                                                             │
     │     if language == "zh-CN":                                                                       │
     │         return {                                                                                  │
     │             'buy': '买入', 'hold': '持有', 'sell': '卖出',                                        │
     │             'BUY': '买入', 'HOLD': '持有', 'SELL': '卖出'                                         │
     │         }                                                                                         │
     │     else:                                                                                         │
     │         return {                                                                                  │
     │             '买入': 'buy', '持有': 'hold', '卖出': 'sell',                                        │
     │             '购买': 'buy', '保持': 'hold', '出售': 'sell'                                         │
     │         }                                                                                         │
     │                                                                                                   │
     │ 阶段3：扩展其他Agent（可选，4-6小时）                                                             │
     │                                                                                                   │
     │ 3.1 修改新闻分析师                                                                                │
     │                                                                                                   │
     │ 文件: D:\projects\TradingAgents-CN-main\tradingagents\agents\analysts\news_analyst.py             │
     │                                                                                                   │
     │ 3.2 修改社交媒体分析师                                                                            │
     │                                                                                                   │
     │ 文件: D:\projects\TradingAgents-CN-main\tradingagents\agents\analysts\social_media_analyst.py     │
     │                                                                                                   │
     │ 3.3 修改中国市场分析师                                                                            │
     │                                                                                                   │
     │ 文件: D:\projects\TradingAgents-CN-main\tradingagents\agents\analysts\china_market_analyst.py     │
     │                                                   