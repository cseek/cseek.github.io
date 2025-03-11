进化笔记

## Debug
```bash
sudo apt update
sudo apt install git
sudo apt install nodejs
sudo apt install npm
sudo apt install ruby-full
sudo apt install build-essential
sudo apt install zlib1g-dev
echo '# Install Ruby Gems to ~/.gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/.gems"' >> ~/.bashrc
echo 'export PATH="$HOME/.gems/bin:$PATH"' >> ~/.bashrc
gem install jekyll bundler
sudo apt install bundler
./tools/init.sh 
./tools/run.sh 
```

## Commit

提交例子
```bash
git commit -m 'feat: hello world'
类型必须是 [build、chore、ci、docs、feat、fix、perf、refactor、revert、style、test] 之一 [type-enum]
```

## Discussion
评论区配置
```bash
https://giscus.app/zh-CN
```