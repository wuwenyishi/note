### QComboBox 的选中时触发事件

在 QComboBox 中，当用户选择下拉列表框中的项时，会触发 `activated` 和 `currentIndexChanged` 两个事件。这两个事件在用户选择不同的项时触发的时机略有不同。

1. `activated(int index)` 事件：当用户从下拉列表框中选择了一个项并完成选择（例如，点击下拉列表框中的项或按下回车键）时触发。参数 `index` 表示用户选择的项在列表中的索引。

```cpp
connect(comboBox, QOverload<int>::of(&QComboBox::activated), [=](int index) {
    // 处理用户选择的项
});
```

2. `currentIndexChanged(int index)` 事件：当用户从下拉列表框中选择了一个项时，即使下拉列表框仍然处于下拉状态，也会触发此事件。参数 `index` 表示用户选择的项在列表中的索引。

```cpp
connect(comboBox, QOverload<int>::of(&QComboBox::currentIndexChanged), [=](int index) {
    // 处理用户选择的项
});
```

根据您的需求，您可以选择连接这两个事件中的一个或两个，以便在用户选择下拉列表框中的项时触发相应的操作或逻辑。

