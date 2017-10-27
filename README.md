# 使用contenteditable+div模拟textarea文本域实现高度自适应
开发过程中由于需要在发送消息的时候需要有一个可以高度自适应的文本域，一开始是使用textarea并搭配auto-size插件来做到textarea的高度自适应，后来因为遇到一些问题，而且也多加了依赖缺乏可定制，所以决定使用contenteditable来实现。

## contenteditable介绍

**contenteditable**属性规定元素内容是否可编辑，是H5新增的属性，支持情况相当好，基本上所有的浏览器都兼容。

**语法：**

```html
<element contenteditable="true|false">
```
## 实现主要代码如下
代码实现是基于vue来实现的。
**html部分：**

```html
<template>
    <div class="textarea" contenteditable="true"></div>
</template>
```
**CSS部分**

```css
<style scoped lang="less" rel="stylesheet/less">
    .textarea {
        box-sizing: border-box;
        min-height: 136px;
        max-height: 300px;
        margin-left: auto;
        margin-right: auto;
        padding: 3px;
        outline: 0;
        border: 1px solid #a0b3d6;
        font-size: 12px;
        word-wrap: break-word;
        overflow-x: hidden;
        overflow-y: auto;
        _overflow-y: visible;
        -webkit-user-modify: read-write-plaintext-only; // 只是编辑text文本，只能解决webkit内核里面问题，手机端适用
        -webkit-user-select: text; // 解决IOS部分手机不支持contenteditable=true属性问题
        p {
            margin: 0;
        }
    }
</style>
```
**代码解读：**

* 设置-webkit-user-modify属性，是为了在剪切复制的时候会把剪切的内容的格式也一并带过来，由于我们是仿写textarea，是不支持富文本的，所以需要需要将内容格式化成文本格式，而该属性在webkit内核下就可以达到我们的目的。
* 设置-webkit-user-select属性，是为了解决在测试过程中出现部分IOS手机不支持contenteditable属性的问题。

**JS部分：**

```js
<script type="text/babel">
    export default {
        mounted() {
            this.addInputEvent();
            this.addFocusEvent();
            this.addEventPaste(this.$el);
        },
        methods: {
            /**
             * 监听鼠标input事件
             */
            addInputEvent(){
                this.$el.addEventListener('input', () => {
                    this.$emit('input', this.getValue());
                })
            },
            /**
             * 监听鼠标获取焦点事件
             */
            addFocusEvent(){
                this.$el.addEventListener('focus', () => {
                    setTimeout(() => {
                        // 解决：如果ios手机使用的不是原生键盘（也可能不止IOS手机有这个问题），则会出现键盘挡住输入框问题，当bottom=0的情况，使用这个属性就可以滚动屏幕中央
                        this.$el.scrollIntoView(true);
                    }, 300);
                    this.$emit('focus');
                })
            },
            /**
             * 追加
             * @param value
             * @param bool
             */
            appendValue(value, bool) {
                this.$el.innerText += value;
                this.$emit('input', this.getValue());
            },
            /**
             * 监听复制事件，去除样式得到纯文本
             */
            addEventPaste: function (el) {
                // 干掉IE http之类地址自动加链接
                try {
                    document.execCommand("AutoUrlDetect", false, false);
                } catch (e) {
                }
                // 监听复制paste事件，目的是为了让-webkit-user-modify属性兼容IE8，毕竟该属性在IE兼容性不好
                el.addEventListener('paste', function (e) {
                    e.preventDefault();
                    var text = null;
                    if (window.clipboardData && clipboardData.setData) {
                        // IE
                        text = window.clipboardData.getData('text');
                    } else {
                        text = (e.originalEvent || e).clipboardData.getData('text/plain') || prompt('在这里输入文本');
                    }
                    // 这里的目的是为了将鼠标的光标移动到复制之后文本的末尾的末尾
                    if (document.body.createTextRange) {
                        if (document.selection) {
                            textRange = document.selection.createRange();
                        } else if (window.getSelection) {
                            sel = window.getSelection();
                            var range = sel.getRangeAt(0);
                            // 创建临时元素，使得TextRange可以移动到正确的位置
                            var tempEl = document.createElement("span");
                            tempEl.innerHTML = "&#FEFF;";
                            range.deleteContents();
                            range.insertNode(tempEl);
                            textRange = document.body.createTextRange();
                            textRange.moveToElementText(tempEl);
                            tempEl.parentNode.removeChild(tempEl);
                        }
                        textRange.text = text;
                        textRange.collapse(false);
                        textRange.select();
                    } else {
                        // Chrome之类浏览器
                        document.execCommand("insertText", false, text);
                    }
                });
            },
            /**
             * 替换
             * @param value
             */
            setValue(value) {
                this.$el.innerText = value;
                this.$emit('input', this.getValue());
            },
            /**
             * 获取值
             * @returns {*}
             */
            getValue() {
                return this.getHtmlToText();
            },
            /**
             * 获取HTML转换之后的文本（去除div标签，替换<br/>为换行）
             * @returns {string}
             */
            getHtmlToText() {
                return this.replaceToBreak(this.getHtml());
            },
            /**
             * 获取HTML
             */
            getHtml() {
                return document.querySelector('.textarea').innerHTML
            },
            /**
             * 替换DIV到换行符
             * @returns {string}
             */
            replaceToBreak(html) {
                html = String(html).replace(/<\/div>/gi, '');
                html = html.replace(/<div>(<br>)?(<br\/>)?/gi, '\n');
                html = html.replace(/<br>|<br\/>/gi, '\n');
                return html;
            },
            /**
             * 获取纯text文本
             * @returns {string}
             */
            getText(){
                if (window.navigator.appName.indexOf("Explorer") > -1)
                    return this.$el.innerText;
                else
                    return this.$el.textContent;
            }
        },
    }
</script>
```
**代码解读：**

* 其中**addEventPaste**方法是为了解决非webkit内核对于**-webkit-user-select**属性支持不好的问题。里面主要是监听黏贴事件然后或者剪切板的文本内容然后再阻止黏贴事件，并将文本内容追加到光标中，并将光标移动到相应的位置。
* 其中**replaceToBreak**方法是为了解决在textarea中换行的问题，在该伪textarea中换行是会单独将换行内容放到新的DIV中的，所以，当我们需要对该内容进行格式化处理才行。
* **this.$el.scrollIntoView**的作用是为了当使用者将我们的输入框是使用绝对定位放在页面底部的时候而被虚拟键盘遮挡的问题。

## issue
* 使用该组件注意一个问题就是不要在可视化区域的节点上使用-**webkit-user-select: none**样式，否则会出现当鼠标焦点小时光标和小水滴无法消失的情况

## github地址
[github项目地址](https://github.com/yejiaming/textarea/tree/master)

## 参考链接

[div模拟textarea文本域轻松实现高度自适应](http://www.zhangxinxu.com/wordpress/2010/12/div-textarea-height-auto/)
[如何让contenteditable元素只能输入纯文本](http://www.zhangxinxu.com/wordpress/2016/01/contenteditable-plaintext-only/)


