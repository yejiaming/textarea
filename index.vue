<template>
    <div class="textarea" contenteditable="true"></div>
</template>
<!--使用该组件注意一个问题就是不要在可视化区域的节点上使用-webkit-user-select: none;样式，否则会出现当鼠标焦点小时光标和小水滴无法消失的情况-->
<script type="text/babel">
    export default {
        data () {
            return {}
        },
        computed: {},
        props: {},
        beforeMount () {
        },
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