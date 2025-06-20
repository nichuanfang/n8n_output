使用Mockito的@Mock、@InjectMocks、@RunWith(PowerMockRunner.class)注解以及Mockito类写单元测试时，提高方法覆盖率和行覆盖率的技巧总结如下：

1. 合理使用@Mock和@InjectMocks
- @Mock用于创建依赖对象的模拟实例，避免真实依赖带来的复杂性和副作用。
- @InjectMocks用于将@Mock创建的模拟对象注入到被测试类中，保证测试环境的隔离。
- 通过模拟依赖，可以专注测试目标方法的逻辑，覆盖更多代码路径。

2. 使用@RunWith(PowerMockRunner.class)
- PowerMockRunner支持对静态方法、final类、私有方法等难以mock的代码进行mock。
- 结合PowerMockito可以mock静态方法、构造函数等，覆盖更多代码分支。

3. 编写全面的测试用例
- 针对不同输入、边界条件、异常情况编写测试用例，覆盖所有代码路径。
- 使用Mockito的when...thenReturn模拟不同返回值，测试各种逻辑分支。

4. 使用Mockito的verify验证交互
- 验证mock对象的方法调用次数和参数，确保代码逻辑正确执行。
- 通过验证交互覆盖代码中的条件判断和分支。

5. 结合代码覆盖率工具
- 使用JaCoCo等覆盖率工具检测测试覆盖率，找出未覆盖代码。
- 针对未覆盖代码补充测试用例，提升覆盖率。

6. 避免过度mock
- 只mock必要的依赖，保持测试的真实性和有效性。
- 过度mock可能导致测试失去意义，反而降低代码质量。

7. 关注异常和边界测试
- 模拟异常抛出，测试异常处理逻辑。
- 测试边界条件，确保代码健壮。

8. 使用参数化测试
- 通过JUnit参数化测试，批量测试多种输入，提高覆盖率。

参考资料：
- CSDN博客《JAVA实战：如何让单元测试覆盖率达到80%甚至以上以及碰到的坑》https://blog.csdn.net/zth_killer/article/details/129693691
- 博客园《使用PowerMockRunner和Mockito编写单元测试用例详解》https://www.cnblogs.com/charlypage/p/14672637.html
- Stack Overflow关于@RunWith(PowerMockRunner.class)的讨论 https://stackoverflow.com/questions/38268929/runwithpowermockrunner-class-vs-runwithmockitojunitrunner-class

总结：合理利用Mockito和PowerMock的注解与API，结合全面的测试用例设计和覆盖率工具，重点测试异常和边界情况，可以有效提升方法覆盖率和行覆盖率。