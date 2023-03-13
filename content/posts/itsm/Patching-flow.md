# Patching flow:

# Jobs list: 任务清单
- Step1:  Release security patch with severity rating:   发布严重等级的安全补丁
- Step2: Provide default patch baseline for Linux and Windows  
- Step3: Check result       
- Step4: Analyze and Reinstall 
- Step5: Analyze and Reinstall in next patching cycle
- Step6: Push to Production env Thursday, 1st week (Month +1) 推送到生产环境，第一周
- Step7: Security Exception Review 安全异常审查

# Status: 状态
   - start
- Emergent Critical 紧急临界
- Fixable                     可修复
- Unfixable                不可修复
- Failure                      失败 
- Sucess                     成功
- End

# Condition： 条件
- If severity rating is High, or Medium for GXP/MLPS app - 如果严重等级为“高”，或者 GXP/MLPS 应用的“中”

# action：动作
- Approve 批准
- Withdraw 提取
- Withdraw patch event 撤销补丁事件
