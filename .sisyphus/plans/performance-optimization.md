# 性能优化计划

## TL;DR

> **Quick Summary**: 低风险、高收益的性能优化，专注于颜色计算和控制台日志
> 
> **Deliverables**:
> - 优化 colorDistance 函数（移除不必要的 Math.sqrt）
> - 移除客户端 console.log
> - 优化 FocusCanvas 颜色查找（Set 替代 .some()）
> - 优化 ColorPanel 渲染（添加 useMemo）
> 
> **Estimated Effort**: Quick
> **Parallel Execution**: YES - 1 wave
> **Critical Path**: Wave 1 → Verification

---

## Context

### Original Request
基于性能分析结果，进行高优先级、保守的性能优化

### Interview Summary
**Key Discussions**:
- 优化范围：仅高优先级
- 改动强度：保守（低风险）
- 包含可选扩展：是（FocusCanvas, ColorPanel）

**Research Findings**:
- colorDistance 函数中 Math.sqrt 不必要（只需比较大小）
- 多处 console.log 影响生产环境性能
- FocusCanvas .some() 可用 Set.has() 替代（O(n) → O(1)）
- ColorPanel 颜色列表可添加 useMemo 避免重复计算

### Metis Review
**Identified Gaps** (addressed):
- 需要同步调整 colorDistance 阈值（150 → 22500）
- 检查 pixelEditingUtils.ts 是否有类似阈值

---

## Work Objectives

### Core Objective
进行低风险、高收益的性能优化，不破坏现有功能

### Concrete Deliverables
- colorDistance 函数优化
- 颜色距离阈值同步调整
- 移除客户端 console.log
- FocusCanvas 颜色查找优化
- ColorPanel 添加 useMemo

### Definition of Done
- [ ] `npm run build` 成功
- [ ] `npm run lint` 无新错误

### Must Have
- 所有改动都是低风险的
- 不破坏现有功能
- 不涉及架构重构

### Must NOT Have (Guardrails)
- 不要使用 Web Worker
- 不要进行组件拆分
- 不要重构状态管理
- 不要删除 console.error 和 console.warn

---

## Verification Strategy

### Test Decision
- **Infrastructure exists**: NO
- **Automated tests**: None
- **Agent-Executed QA**: `npm run build` + `npm run lint`

### QA Policy
所有优化完成后运行构建和 lint 验证。

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Start Immediately — 全部并行):
├── Task 1: 优化 colorDistance 函数 [quick]
├── Task 2: 调整颜色距离阈值 [quick]
├── Task 3: 移除客户端 console.log [quick]
├── Task 4: 优化 FocusCanvas 颜色查找 [quick]
└── Task 5: 优化 ColorPanel 组件 [quick]

Wave FINAL (After Wave 1 — verification):
├── Task F1: 构建验证 [quick]
└── Task F2: Lint 验证 [quick]
```

### Agent Dispatch Summary
- **All tasks**: `quick` category
- **Total files to modify**: 5
- **Estimated code changes**: ~30 lines

---

## Technical Debt (记录，不实施)

以下优化在本次计划中不做，但值得记录：

1. **Web Worker 迁移** (高收益，中等风险)
   - 将像素化处理移到 Web Worker
   - 将洪水填充算法移到 Web Worker
   - 预期提升：主线程阻塞减少 80%

2. **Immer 替代深拷贝** (中等收益，低风险)
   - 使用 Immer 替代 JSON.parse(JSON.stringify())
   - 预期提升：深拷贝速度提升 2-5x

3. **增量历史记录** (中等收益，中等风险)
   - 只存储变更的 diff 而非完整快照
   - 预期提升：内存使用减少 50%

4. **虚拟化 Canvas 渲染** (高收益，高风险)
   - 只渲染可见区域的像素
   - 预期提升：大网格渲染帧率提升 3-5x

---

## TODOs

- [x] 1. 优化 colorDistance 函数

  **What to do**:
  在 `src/utils/pixelation.ts` 中优化 `colorDistance` 函数，移除不必要的 `Math.sqrt`。\n
  由于颜色距离只用于比较大小（找最近颜色），开平方是多余的。
---

- [x] 2. 调整颜色距离阈值

  **What to do**:
  由于 `colorDistance` 现在返回平方距离，需要同步调整相关的阈值比较:
 
  文件 1: `src/utils/pixelation.ts`
  - Line 74: `if (blackDistance < 150)` → `if (blackDistance < 22500)` (150² = 22500)
  - Line 83: `if (whiteDistance < 150)` → `if (whiteDistance < 22500)`

  文件 2: `src/utils/pixelEditingUtils.ts`
  - 检查是否有类似的距离阈值并调整

---

- [ ] 3. 移除客户端 console.log

  **What to do**:
  移除以下文件中的 `console.log` 语句（保留 `console.error` 和 `console.warn`）：
  
  1. `src/utils/pixelation.ts` - 搜索并移除
  2. `src/utils/localStorageUtils.ts` - 搜索并移除
  3. `src/hooks/usePixelEditingOperations.ts` - 搜索并移除
  4. `src/components/PatternEditor.tsx` - 搜索并移除客户端日志

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1-2)
  - **Blocks**: None
  - **Blocked By**: None

  **References**:
  - `src/utils/pixelation.ts`
  - `src/utils/localStorageUtils.ts`
  - `src/hooks/usePixelEditingOperations.ts`
  - `src/components/PatternEditor.tsx`

  **Acceptance Criteria**:
  - [ ] 上述文件中的 console.log 已移除
  - [ ] console.error 和 console.warn 保留

  **Commit**: YES
  - Message: `perf: remove client-side console.log statements`
  - Files: 上述文件

---

- [ ] 4. 优化 FocusCanvas 颜色查找

  **What to do**:
  在 `src/components/FocusCanvas.tsx` 中，将 `.some()` 查找优化为 Set 查找。
  
  当前 `.some()` 是 O(n)，Set.has() 是 O(1)。

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1-3)
  - **Blocks**: None
  - **Blocked By**: None

  **References**:
  - `src/components/FocusCanvas.tsx:108-110` - .some() 使用位置

  **Acceptance Criteria**:
  - [ ] remainingColorsSet 已创建
  - [ ] .some() 已替换为 .has()

  **Commit**: YES
  - Message: `perf: optimize FocusCanvas color lookup with Set`
  - Files: src/components/FocusCanvas.tsx

---

- [ ] 5. 优化 ColorPanel 组件

  **What to do**:
  在 `src/components/ColorPanel.tsx` 中添加 `useMemo` 优化颜色列表渲染。

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1-4)
  - **Blocks**: None
  - **Blocked By**: None

  **References**:
  - `src/components/ColorPanel.tsx:27-45` - 颜色列表渲染

  **Acceptance Criteria**:
  - [ ] 颜色列表使用 useMemo 缓存
  - [ ] 渲染时不创建不必要的临时对象

  **Commit**: YES
  - Message: `perf: add useMemo to ColorPanel color list`
  - Files: src/components/ColorPanel.tsx

---

## Final Verification Wave

- [ ] F1. **构建验证**

  **What to do**:
  运行 `npm run build` 确保所有优化没有破坏构建

  **Acceptance Criteria**:
  - [ ] `npm run build` 成功无错误

  **QA Scenario**:
  ```
  Scenario: 构建成功
    Tool: Bash
    Steps:
      1. npm run build
    Expected Result: Build completed successfully
    Evidence: .sisyphus/evidence/build-success.log
  ```

- [ ] F2. **Lint 验证**

  **What to do**:
  运行 `npm run lint` 确保代码质量

  **Acceptance Criteria**:
  - [ ] `npm run lint` 无新错误（警告可接受）

  **QA Scenario**:
  ```
  Scenario: Lint 通过
    Tool: Bash
    Steps:
      1. npm run lint
    Expected Result: No new errors
    Evidence: .sisyphus/evidence/lint-success.log
  ```

---

## Commit Strategy

- **Commit 1**: `perf: optimize colorDistance by removing unnecessary sqrt` (Task 1-2)
- **Commit 2**: `perf: remove client-side console.log statements` (Task 3)
- **Commit 3**: `perf: optimize FocusCanvas color lookup with Set` (Task 4)
- **Commit 4**: `perf: add useMemo to ColorPanel color list` (Task 5)

---

## Success Criteria

### Verification Commands
```bash
npm run build  # Expected: Build completed successfully
npm run lint   # Expected: No new errors
```

### Final Checklist
- [ ] 所有优化都是低风险改动
- [ ] 不涉及架构重构
- [ ] 构建验证通过
- [ ] Lint 验证通过
- [ ] 性能提升（预期：颜色计算速度提升 30-50%）
