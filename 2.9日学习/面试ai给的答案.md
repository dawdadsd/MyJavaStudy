以下是针对您提出的问题逐一进行的详细解答和修正：

---

### **1. 基础概念类问题**
**Q**: HashMap和ArrayList的底层数据结构是什么？默认初始容量？

**修正答案**：
- **HashMap**: 
  - **数据结构**：数组 + 链表/红黑树（JDK8+）。数组中的每个元素称为“桶”（bucket），每个桶可能存储链表节点（`Node`）或红黑树节点（`TreeNode`）。
  - **默认初始容量**：16。
- **ArrayList**:
  - **数据结构**：动态数组（`Object[] elementData`）。
  - **默认初始容量**：10。

---

### **2. 扩容机制类问题**
**Q**: HashMap和ArrayList的扩容条件及扩容后大小？

**修正答案**：
- **HashMap**:
  - **触发条件**：当元素数量超过 `容量 × 负载因子`（默认负载因子0.75，容量16时阈值为12）。
  - **扩容大小**：容量翻倍（如16 → 32）。
- **ArrayList**:
  - **触发条件**：当添加元素时发现数组已满（`size == elementData.length`）。
  - **扩容大小**：新容量为原容量的1.5倍（如10 → 15）。

---

### **3. 性能优化类问题**
**Q**: 存储100万条数据时如何优化初始容量？

**修正答案**：
- **HashMap**:
  - 设置初始容量为 `(预期元素数 / 负载因子) + 1`。例如，100万数据需设置初始容量为 `1000000 / 0.75 ≈ 1333334`，取最近的2的幂（2^20=1,048,576，可能不足，需设为2^21=2,097,152）。
- **ArrayList**:
  - 直接设置初始容量为100万（`new ArrayList<>(1_000_000)`），避免多次扩容。

---

### **4. 哈希冲突类问题**
**Q**: HashMap如何解决哈希冲突？为什么链表长度≥8转红黑树？

**修正答案**：
- **解决冲突方法**：链地址法（链表）、红黑树（JDK8+）。
- **转换阈值**：
  - 链表长度≥8时转为红黑树（查找时间复杂度从O(n)优化为O(log n)）。
  - 红黑树节点数≤6时转回链表。
  - **选择8的原因**：基于泊松分布统计，哈希冲突达到8的概率极低（约0.000006%），避免不必要的树化开销。

---

### **5. 实践应用类问题**
**Q**: 大量数据频繁随机访问，选HashMap还是ArrayList？

**修正答案**：
- **选择ArrayList**：
  - 数组支持O(1)随机访问（通过索引）。
  - HashMap的随机访问需要计算哈希值，可能因哈希碰撞退化为O(log n)或O(n)。
- **例外场景**：若需要通过键（key）快速查找值（value），则选择HashMap。

---

### **6. 源码分析类问题**
**Q**: HashMap的put方法流程及重复key处理逻辑？

**修正答案**：
1. 计算key的哈希值（`hash(key)`）。
2. 若数组未初始化，调用`resize()`扩容。
3. 计算桶索引：`(n-1) & hash`。
4. 若桶为空，直接插入新节点。
5. 若桶不为空：
   - 链表：遍历查找相同key，存在则更新value；否则插入链表尾部。
   - 红黑树：调用树节点的插入方法。
6. 若链表长度≥8，转为红黑树。
7. 检查是否扩容。

---

### **7. 底层原理类问题**
**Q**: 为什么HashMap容量是2的幂？

**修正答案**：
- **索引计算优化**：通过 `(n-1) & hash` 替代 `hash % n`，位运算效率更高。
- **均匀分布**：2的幂能保证低位掩码（n-1）的二进制全为1，减少哈希冲突。

---

### **8. 进阶特性问题**
**Q**: 负载因子的作用及默认0.75的原因？

**修正答案**：
- **负载因子**：权衡时间与空间的参数。值越小，扩容频繁（空间占用多，哈希冲突少）；值越大，扩容延迟（空间少，哈希冲突多）。
- **默认0.75**：经验值，在时间和空间成本之间取得平衡。

---

### **9. 异常处理问题**
**Q**: HashMap可能遇到的异常及避免方法？

**修正答案**：
- **ConcurrentModificationException**：遍历时修改集合（如迭代中删除元素）。
  - **避免方法**：使用迭代器的`remove()`方法或`ConcurrentHashMap`。
- **哈希碰撞攻击**：恶意构造大量相同哈希的key导致性能下降。
  - **避免方法**：使用`String`等不可变对象作为key，或自定义哈希函数。

---

### **10. 对比类问题**
**Q**: HashMap与Hashtable的区别？

**修正答案**：
- **线程安全**：Hashtable是线程安全的（方法用`synchronized`修饰），HashMap非线程安全。
- **性能**：HashMap性能更高（无锁竞争）。
- **Null支持**：HashMap允许key和value为null，Hashtable不允许。
- **使用场景**：需线程安全且不介意性能损耗时选Hashtable；否则用`ConcurrentHashMap`替代。

---

### **11. 实现细节问题**
**Q**: 为何不建议可变对象作为HashMap的key？

**修正答案**：
- **问题**：若对象状态变化导致`hashCode()`改变，无法通过原key检索到条目。
- **注意事项**：
  - 确保对象不可变。
  - 若必须可变，需确保状态变化不影响`hashCode()`和`equals()`。

---

### **12. 性能问题**
**Q**: 如何判断HashMap性能问题及优化手段？

**优化手段**：
1. **监控哈希冲突**：检查链表长度是否频繁≥8。
2. **调整初始容量和负载因子**：避免多次扩容。
3. **优化hashCode()**：确保分布均匀。
4. **使用专用集合**：如`LinkedHashMap`保持顺序，`ConcurrentHashMap`应对并发。

---

### **总结**
- **HashMap**：适合键值查找，需关注哈希冲突和扩容策略。
- **ArrayList**：适合顺序访问和随机索引操作，需预分配容量避免扩容开销。
- **关键优化点**：合理设置初始容量、理解数据结构特性、避免常见异常。