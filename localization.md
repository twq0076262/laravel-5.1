# 本地化

## 1、简介
Laravel 的本地化特性提供了一个方便的方式从多个语言文件中获取字符串，从而允许你在应用中轻松支持多种语言。

语言字符串存放在 `resources/lang` 目录中，在该目录中应该包含应用支持的每种语言的子目录：

```
/resources
    /lang
        /en
            messages.php
        /es
            messages.php
```

所有语言文件都返回一个键值对数组，例如：

```
<?php

return [
    'welcome' => 'Welcome to our application'
];
```

### 1.1 配置 Locale 选项
应用默认语言存放在配置文件 `config/app.php` 中，当然，你可以修改该值来匹配应用需要。你还可以在运行时使用 `App` 门面上的 `setLocale` 方法改变当前语言：

```
Route::get('welcome/{locale}', function ($locale) {
    App::setLocale($locale);

    //
});
```

你还可以配置一个“备用语言”，当当前语言不包含给定语言行时备用语言被返回。和默认语言一样，备用语言也在配置文件 `config/app.php` 中配置：

```
'fallback_locale' => 'en',
```

## 2、基本使用
你可以使用帮助函数 `trans` 从语言文件中获取行，该方法接收文件和语言行的键作为第一个参数，例如，让我们在语言文件 `resources/lang/messages.php` 中获取语言行 `welcome`：

```
echo trans('messages.welcome');
```

当然如果你使用 Blade 模板引擎，可以使用{{ }}语法打印语言行：

```
{{ trans('messages.welcome') }}
```

如果指定的语言行不存在，`trans` 函数将返回语言行的键，所以，使用上面的例子，如果语言行不存在的话，`trans` 函数将返回 `messages.welcome`。

**替换语言行中的参数**
如果需要的话，你可以在语言行中定义占位符，所有的占位符都有一个:前缀，例如，你可以用占位符名称定义一个 `welcome` 消息：

```
'welcome' => 'Welcome, :name',
```

要在获取语言行的时候替换占位符，传递一个替换数组作为 `trans` 函数的第二个参数：

```
echo trans('messages.welcome', ['name' => 'Dayle']);
```

### 2.1 多元化
多元化是一个复杂的问题，因为不同语言对多元化有不同的规则，通过使用管道字符“|”，你可以区分一个字符串的单数和复数形式：

```
'apples' => 'There is one apple|There are many apples',
```

然后，你可以使用 `trans_choice` 函数获取给定行数的语言行，在本例中，由于行数大于 1，将会返回语言行的复数形式：

```
echo trans_choice('messages.apples', 10);
```

由于 Laravel 翻译器由 Symfony 翻译组件提供，你可以创建更复杂的多元化规则：

```
'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',
```

## 3、覆盖 Vendor 包中语言文件
有些包可以处理自己的语言文件。你可以通过将自己的文件放在 `resources/lang/vendor/{package}/{locale}`目录下来覆盖它们而不是破坏这些包的核心文件来调整这些句子。
所以，举个例子，如果你需要覆盖名为 `skyrim/hearthfire` 的包中的 `messages.php` 文件里的英文句子，可以创建一个 `resources/lang/vendor/hearthfire/en/messages.php` 文件。在这个文件中只需要定义你想要覆盖的句子，你没有覆盖的句子仍然从该包原来的语言文件中加载。