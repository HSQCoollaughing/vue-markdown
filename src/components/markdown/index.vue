<template>
    <div id="markdowm">
        <div class="md-title">
            <ul class="cf">
                <li>
                    <span>插入图片</span>
                    <input type="file" class="uploadFile" @change="insertImg">
                </li>
                <li @click="maskBol=true">
                    <span>插入链接</span>
                </li>
                <li @click="insertCode">
                    <span>代码块</span>
                </li>
                <li @click="setCursorPosition($refs.text, '***')">
                    <span>分割线</span>
                </li>
                <li @click="setCursorPosition($refs.text, '****', 2)">
                    <span>粗体</span>
                </li>
                <li @click="setCursorPosition($refs.text, '**', 1)">
                    <span>斜体</span>
                </li>
                <li @click="setCursorPosition($refs.text, '> ', 2)">
                    <span>引用</span>
                </li>
            </ul>
        </div>
        <textarea v-model="val" @input="handleModelInput" ref="text" @keydown.tab="tabMarkdown"></textarea>
        <div class="render fmt" v-html="renderHtml"></div>

        <div class="mask" v-show="maskBol">
            <div class="link-text">
                <input type="text" v-model="link">
                <button @click="insertLink">确定</button>
            </div>
        </div>
        
    </div>
</template>

<script>
import marked from 'marked'
import highlightJs from 'highlight.js'
    export default {
        props: ['value'],
        data() {
            return {
                val: this.value,
                link: '',
                maskBol: false,
                textarea: null
            }
        },
        mounted () {
            this.textarea = this.$refs.text
        },
        methods: {
            tabMarkdown (e) { // tab键
                e.preventDefault()
                let indent = '    '
                let start = this.textarea.selectionStart
                let end = this.textarea.selectionEnd
                let selected = window.getSelection().toString()
                selected = indent + selected.replace(/\n/g, '\n' + indent)
                this.textarea.value = this.textarea.value.substring(0, start) + selected
                        + this.textarea.value.substring(end);
                this.textarea.setSelectionRange(start + indent.length, start
                        + selected.length)
            },
            handleModelInput() {
                this.$emit('input', this.val)
            },
            async insertImg(e) { // 插入图片
                let formData = new FormData(),
                    img = '';
                formData.append('img', e.target.files[0]);
                try {
                    let data = await this.axios({
                        method: 'post',
                        url: 'http://localhost:3000/markdown_upload_img',
                        data: formData
                    })
                    img = data.data.img
                } catch (e) {
                    console.log(e)
                }
                
                let val = `![图片描述](${img})`
                this.setCursorPosition(this.$refs.text, val, 6)
            },
            insertLink () { //插入链接
                this.maskBol = false
                let val = `[链接描述](${this.link})`
                this.setCursorPosition(this.$refs.text, val, 5)
            },
            insertCode () {
                let val = `
\`\`\`
    
\`\`\``
                this.setCursorPosition(this.$refs.text, val, val.length-8)
            },
            setCursorPosition (dom,val,posLen) { // 设置光标位置
                var cursorPosition = 0;
                if(dom.selectionStart){
                    cursorPosition = dom.selectionStart;
                }
                this.insertAtCursor(dom,val);
                dom.focus();
                dom.setSelectionRange(dom.value.length,cursorPosition + (posLen || val.length));
                this.val = dom.value
            },
            insertAtCursor(dom, val) { // 光标所在位置插入字符
                if (document.selection){
                    dom.focus();
                    sel = document.selection.createRange();
                    sel.text = val;
                    sel.select();
                }else if (dom.selectionStart || dom.selectionStart == '0'){
                    let startPos = dom.selectionStart;
                    let endPos = dom.selectionEnd;
                    let restoreTop = dom.scrollTop;
                    dom.value = dom.value.substring(0, startPos) + val + dom.value.substring(endPos, dom.value.length);
                    if (restoreTop > 0){
                        dom.scrollTop = restoreTop;
                    }
                    dom.focus();
                    dom.selectionStart = startPos + val.length;
                    dom.selectionEnd = startPos + val.length;
                } else {
                    dom.value += val;
                    dom.focus();
                }
            }
        },
        computed: {
            renderHtml() {
                marked.setOptions({
                    renderer: new marked.Renderer(),
                    gfm: true, //允许 Git Hub标准的markdown.
                    tables: true, //允许支持表格语法。该选项要求 gfm 为true。
                    breaks: true, //允许回车换行。该选项要求 gfm 为true。
                    pedantic: false, //尽可能地兼容 markdown.pl的晦涩部分。不纠正原始模型任何的不良行为和错误。
                    sanitize: true, //对输出进行过滤（清理），将忽略任何已经输入的html代码（标签）
                    smartLists: true, //使用比原生markdown更时髦的列表。 旧的列表将可能被作为pedantic的处理内容过滤掉.
                    smartypants: false, //使用更为时髦的标点，比如在引用语法中加入破折号。
                    highlight: function (code) {
                        return highlightJs.highlightAuto(code).value;
                    }
                });
                return marked(this.val)
            }
        }
    }
</script>

<style lang="less" scoped>
    @import './markdown.css';
    @md-bd-color: #ccc;
    @md-title-color: #EDF2FC;
    @md-bg-color: #E6EBF5;
    @btn-hover: #409eff;
    #markdowm {
        width: 100%;
        height: 500px;
        text-align: left;
        overflow: hidden;
        border: 1px solid @md-bd-color;
        position: relative;
        .md-title {
            width: 100%;
            height: 60px;
            background: @md-title-color;
            position: absolute;
            left:0;top:0;
            li {
                width: 100px;
                height: 100%;
                line-height: 60px;
                text-align: center;
                position: relative;
                float: left;
                cursor: pointer;
                &:hover {
                    color: @btn-hover;
                }
                &:after {
                    content: '';
                    position: absolute;
                    left: 100%;top:50%;
                    transform: translateY(-50%);
                    width: 1px;
                    height: 20px;
                    background: #ccc;
                }
                .uploadFile {
                    position: absolute;
                    top: 0;
                    left: 0;
                    width: 100%;
                    height: 100%;
                    opacity: 0;
                    cursor: pointer;
                }
            }
        }
        textarea,
        .render {
            float: left;
            width: 50%;
            height: 100%;
            vertical-align: top;
            box-sizing: border-box;
            padding: 0 20px;
        }
        textarea {
            border: none;
            border-right: 1px solid @md-bd-color;
            resize: none;
            outline: none;
            background-color: #f6f6f6;
            font-size: 14px;
            font-family: 'Monaco', courier, monospace;
            padding: 20px;
            padding-top: 60px;
        }
        .render {
            background-color: @md-bg-color;
            overflow-y: scroll;
            padding-top: 60px;
        }

        .mask {
            position: fixed;
            left: 0;top: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, .5);
        }
        .link-text {
            width: 100%;
            text-align: center;
            position: absolute;
            left: 50%;top: 50%;
            transform: translate(-50%, -50%);
            button {
                display: inline-block;
                line-height: 1;
                white-space: nowrap;
                cursor: pointer;
                border: 1px solid #b3d8ff;
                -webkit-appearance: none;
                text-align: center;
                box-sizing: border-box;
                outline: none;
                transition: .1s;
                font-weight: 500;
                -webkit-user-select: none;
                padding: 12px 20px;
                font-size: 14px;
                border-radius: 4px;
                color: #409eff;
                transition: all .3s;
                background: #ecf5ff;
                &:hover{
                    background: #409eff;
                    border-color: #409eff;
                    color: #fff;
                }
            }
            input {
                width: 300px;
                background-color: #fff;
                background-image: none;
                border-radius: 4px;
                border: 1px solid #dcdfe6;
                box-sizing: border-box;
                color: #606266;
                display: inline-block;
                font-size: inherit;
                height: 40px;
                line-height: 1;
                outline: none;
                padding: 0 15px;
                transition: border-color .2s cubic-bezier(.645,.045,.355,1);
                &:focus{
                    outline: none;
                    border-color: #409eff;
                }
            }
        }
    }
</style>