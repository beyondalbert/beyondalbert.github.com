---
layout: post
title: "RSpec测试rails之--如何使用factory_girl创建对象"
description: ""
category: rails笔记
tags: [rails, ruby, test]
---
{% include JB/setup %}

## 使用factory_girl创建对象

基于版本： fancory_girl_rails: 4.5.0

### factory_girl是什么？

* A Replacement for Fixtures
* Provides a Simple DSL
* Keeps Tests Focused & Readable
* Builds Objects Instead of Database Records

### 创建一个factory

一般一个model对象对应一个factories目录下的文件，比如有一个项目model，则建立以下factories文件：
Rails.root/spec/factories/projects.rb

#### 一个最简单的factory:

```ruby
# Rails.root/spec/factories/projects.rb
FactoryGirl.define do
  factory :project do
    name "test project"
    description "test descriprion"
  end
end
```

#### 调用如下：

```ruby
@project = FactoryGirl.create(:project)
# 或者：
@project = FactoryGirl.build(:project)
```

### 重复创建不同内容的对象

```ruby
FactoryGirl.define do
  sequence :name do |n|
    "test#{n} project"
  end
  factory :project do
    name
    description "test descriprion"
  end
end

@project = FactoryGirl.create_list(:project, 5)
# => test1_project
# => test2_project
# => .
# => .
# => .
```

### 重复利用对象的现有attributes

#### 使用trat实现

```ruby
FactoryGirl.define do
  factory :project do
    name "test project"
    description "test descriprion"
  end
  
  trait :with_archived do
    is_archived true
  end
end

# 调用
project = FactoryGirl.create(:project, :with_archived)
project.is_archived # => true
```

#### 使用继承的方式实现

```ruby
FactoryGirl.define do
    factory :project do
        name "test project"
        description "test description"
        
        factory :archived_project do
            is_archived true
        end
    end
end
# 或者：
FactoryGirl.define do
    factory :project do
        name "test project"
        description "test description"
    end
    factory :archived_project, parent: :project do
        is_archived true
    end
end

# 调用：
archived_project = FactoryGirl.create(:archived_project)
archived_project.is_archived # => true
```

### 创建多个相互关联的对象

#### 直接添加

```ruby
# Rails.root/spec/factories/versions.rb
FactoryGirl.define do
  factory :version do
    project
  end
end

# 在创建version时，会自动创建version关联的project,前提是已经定义好了project的FactoryGirl
version = FactoryGirl.create(:version)
```

#### 通过callback实现

```ruby
FactoryGirl.define do
  factory :project do
    name "test project"
  end
  
  trait :with_version do
    after :create do |project|
      FactoryGirl.create :version, project: project
    end
  end
end

# 调用
project = FactoryGirl.create(:project, :with_version)
```

### 使用transient block来自定义创建的对象内容

```ruby
# Rails.root/spec/factories/projects.rb
FactoryGirl.define do
  factory :project do
    transient do
      name_flag false
    end
    name { name_flag ? "test project with flag" : "test project" }
  end
  trait :with_versions do
    transient do
      number_of_versions 3
    end
    after :create do |project, evaluator|
      FactoryGirl.create_list :version, evaluator.number_of_versions, project: project
    end
  end
end

# 调用如下：
project = FactoryGirl.create(:project, name_flag: true)
project.name # => "test project with flag"
FactoryGirl.create(:project, :with_versions, number_of_versions: 5, name_flag: true)
```

#### 参考链接

http://www.slideshare.net/gabevanslv/factory-girl-15924188
http://arjanvandergaag.nl/blog/factory_girl_tips.html
http://www.rubydoc.info/gems/factory_girl/file/GETTING_STARTED.md