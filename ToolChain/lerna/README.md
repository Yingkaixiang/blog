```bash
# 使用统一版本管理的模式
# 即所有的 packages 法布时都为同一个版本
lerna init

# 使用独立的版本管理模式
# 即每个 package 都有一个独立的版本
lerna init --independent

# 向 pkgA 添加第三方依赖如 lodash
# 前提是当前目录已有 package.json 文件
lerna add lodash --scope=pkgA

# 向 pkgA 添加当前项目下的其他 package 依赖
# 如添加 pkgB 到 pkgA
# 使用 symlink 进行 关联
# 而不是我们常用的 npm link
lerna add pkgB --scope=pkgA

# 发布当前工程下全部的 package
# independent 模式会根据每个包不同的版本
# 进行版本号升级
lerna publish

# 如果需要发布公共的 @scope
# 则需要在每一个项目下添加
"publishConfig": {
    "access": "public"
}

# 显示当前更新的 package 以及
# 依赖更新 package 的 package
lerna updated

# 执行所有 package 下的指定脚本
# 如果当前 package 没有此脚本
# 则不会执行
lerna run test
```