# 鼠须管(squirrel)使用全套英文标点

## 配置方法

- 下载一个 yaml 文件

  - [**alternative.yaml**](https://gist.github.com/lotem/2334409)

- 修改 alternative.yaml 文件的 `config_version: "0.3"` 中的版本号为自定义配置文件版本号

- 在自定义配置文件中打 patch

  - 我的自定义文件为 `default.custom.yaml`
  - 在该文件末尾添加以下配置

  ```
  patch:
    "punctuator":
      "import_preset": alternative
  ```

- 重新部署

  - Deploy

## 链接

- [CustomizationGuide · rime/home Wiki](https://github.com/rime/home/wiki/CustomizationGuide#%E4%BD%BF%E7%94%A8%E5%85%A8%E5%A5%97%E8%A5%BF%E6%96%87%E6%A8%99%E9%BB%9E)