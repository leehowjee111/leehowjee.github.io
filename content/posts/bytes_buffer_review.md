+++
date = '2026-04-23T19:35:41+08:00'
draft = false
title = 'bytes_buffer_review'
+++
# 对于bytes_buffer.hpp文件的复盘
##### bytes.buffer.hpp文件分为bytes_buffer,bytes_view,bytes_const_view,static_bytes_buffer。接下来我会将我从中学到的知识进行整理，我会用我的语言进行描述。

### Q1:视图是什么
对于一本书来说，它本来属于图书馆，你只是拥有读的权限，这跟视图的描述是差不多的。
视图就是不拥有数据，只拥有与其指向数据的指针或引用，也就是只能读。
从此就可以看出，视图拥有零拷贝和轻量的特点。
项目中多次使用视图，依次来减少使用产生的拷贝

### Q2:什么是移动构造
说到它，就必须提及拷贝构造
移动构造：必须满足两个条件：1.必须是右值。2.必须是临时对象。
这样，当使用移动构造时，新对象new出来的指针会指向快要消除的临时对象，无需进行开辟空间和拷贝。
```cpp
bytes_buffer(bytes_buffer &&) =default;
bytes_buffer &operator=(bytes_buffer &&) =default;
```
拷贝构造：新对象必须开辟一块内存，然后将临时对象的内容拷贝一份赋值到新对象里。

### Q3:隐式自动转换类型
```cpp
    operator bytes_const_view() const noexcept {
        return bytes_const_view{m_data.data(), m_data.size()};
    }

    operator bytes_view() noexcept {
        return bytes_view{m_data.data(), m_data.size()};
    }

    operator std::string_view() const noexcept {
        return std::string_view{m_data.data(), m_data.size()};
    }

```
operator的使用，可以用来重载运算符操作，这是我当前遇到过的第二种操作，自动转换类型。
当bytes_buffer想要转换为其他类型时，通过这三个可以实现隐式转换。
这个操作相当于创建了一个临时对象，将该对象的内容移动给左值对象，无拷贝
### Q4:append_literial(char const (&literial)[N])的分析
char const (&literial)[N]函数模板 当传入一个字符串时，会根据字符串长度推导出N的值，也就是说不同的参数会创建不同的函数实例。其中&literial使用引用可以防止数组退化成为指针。**这里使用这个的原因是要去除字符串末尾的'\0'**