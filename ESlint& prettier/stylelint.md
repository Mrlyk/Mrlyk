# stylelint



配置示例：

```json
{
  "extends": "stylelint-config-standard",
  "rules": {
    "indentation": 4,
    "no-descending-specificity": null,
    "rule-empty-line-before": null,
    "font-family-no-missing-generic-family-keyword": null // 关闭通用字体检查

  },
  "ignoreFiles": ["public/**/*.css"]
}
```

## 规则配置

- 禁用规则：`null`
- 