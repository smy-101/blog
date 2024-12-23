---
  title: VS Code + Ruby on Rails配置
  pubDate: 2024-12-23
  categories: [后端,Ruby,Rails]
  description: 'VS Code开发Rails的配置介绍'
---

>  为了能更好的在 vs code 上开发 rails，我参考网上的资料进行了一定的配置

## 参考

-  [A decent VS Code + Ruby on Rails setup](https://railsnotes.xyz/blog/vscode-rails-setup)
- [RailsNotes Ruby on Rails Extension Pack](https://marketplace.visualstudio.com/items?itemName=RailsNotes.railsnotes-ruby-on-rails-extension-pack)

## 插件安装

通过安装 RailsNotes Ruby on Rails Extension Pack 这个插件，基本上将所有需要的插件都安装了，有些 lint 插件可以选择使用，这里我使用了 Rubocop 插件放弃了 Standard 插件和 Rufo 插件

## VS Code 设置

```
"solargraph.diagnostics": true,
"solargraph.completion": true,
"solargraph.hover": true,

"rubyLsp.rubyVersionManager": {
	"identifier": "rbenv"
},
"[ruby]": {
	"editor.defaultFormatter": "rubocop.vscode-rubocop"
},
"rubyLsp.formatter": "auto",
```

## Gem 安装

```
group :development, :test do
  gem "ruby-lsp-rails"
  gem "solargraph"
end
```

```
bundle install
```
