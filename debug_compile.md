# 编译错误调试指南

编译错误信息被截断，需要查看完整错误信息。

## 方法1: 增加错误输出限制

```bash
bazel build //tensorflow_recommenders_addons/dynamic_embedding/core:_redis_table_ops.so \
    --verbose_failures \
    --experimental_ui_max_stdouterr_bytes=10485760 \
    --subcommands
```

## 方法2: 直接查看编译命令的输出

```bash
# 先清理构建
bazel clean

# 重新构建并保存输出到文件
bazel build //tensorflow_recommenders_addons/dynamic_embedding/core:_redis_table_ops.so \
    --verbose_failures 2>&1 | tee compile_output.log

# 查看文件最后部分
tail -200 compile_output.log
```

## 方法3: 单独编译 Redis 相关目标

```bash
bazel build //tensorflow_recommenders_addons/dynamic_embedding/core/kernels/redis_table_op.cc \
    --verbose_failures \
    --copt=-v
```

## 常见问题检查清单

1. **Redis++ 依赖**: 确保 `@redis-plus-plus` 和 `@hiredis` 已正确配置
2. **TensorFlow 版本兼容性**: 检查 TensorFlow 版本是否匹配
3. **编译器版本**: 检查 GCC 版本是否符合要求 (需要支持 C++14)
4. **缺少头文件**: 检查是否有未包含的头文件
5. **链接错误**: 检查是否缺少链接库

## 检查 BUILD 文件依赖

当前的 `_redis_table_ops.so` 依赖项：
- `@hiredis`
- `@redis-plus-plus//:redis-plus-plus`
- TensorFlow 相关依赖（通过 `custom_op_library` 自动添加）

如果错误是关于缺少符号或未定义的引用，可能需要添加额外的依赖。

