---
title: �޸����вֿ��Զ�̵�ַ
category: [���ü���]
tags: [git]
---

> ��ʱ����ҪǨ�����еĴ���ֿ⣬������´����²ֿ⣬ɾ����ԭ���� .git Ŀ¼�����ʹ�����ȥ��ԭ�е� commit ��Ϣ��ȫ����ʧ��ʹ���������ַ����Ϳ�����������ԭ�е� commit ��Ϣ��
{: .prompt-info }

## ���뱾�زֿ�Ŀ¼
```bash
cd /path/to/your/local/repo
```

## �鿴��ǰԶ�̵�ַ

```bash
git remote -v
```

## �޸�Զ�̵�ַ

```bash
git remote set-url origin git@172.29.220.100:uav/gzslamB/GZ_SLAM_Middleware.git
```

## ǿ���������з�֧�ͱ�ǩ

```bash
git push --force --all origin
git push --force --tags origin
```

## ע��
��ȷ����Ҫ���õ� URL ���ڵ�Զ�ֿ̲�Ϊ�գ�����ᱻ���ǵ���