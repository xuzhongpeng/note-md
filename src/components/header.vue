<template>
  <div>
    <div class="header">
      <span v-for="(item,index) in actionBtnList" :key="index" class="center col1" span="1">
        <div @click="insert(item.template, item.isSpecial, item.bothEnds)" class="m_action_item">
          <Tooltip :content="item.desc">
            <span class="iconfont" :class="item.icon"></span>
          </Tooltip>
        </div>
      </span>
      <span class="center col" span="1">
        <Dropdown class='' @on-click="goToGithub">
          <a href="javascript:void(0)">
            快捷键
            <Icon type="ios-arrow-down"></Icon>
          </a>
          <DropdownMenu slot="list">
            <DropdownItem>
              ctrl/⌘+2 = H2标题
            </DropdownItem>
            <DropdownItem>
              ctrl/⌘+3 = H3标题
            </DropdownItem>
            <DropdownItem disabled>
              快捷键不支持其他等级标题
            </DropdownItem>
            <DropdownItem>
              ctrl/⌘+b = 加粗
            </DropdownItem>
            <DropdownItem>
              ctrl/⌘+k = 空链接
            </DropdownItem>
            <DropdownItem>
              ctrl/⌘+u = 上传图片
            </DropdownItem>
            <DropdownItem>
              ctrl/⌘+i = 行内代码
            </DropdownItem>
            <DropdownItem>
              ctrl/⌘+shift+i = 代码块
            </DropdownItem>
            <DropdownItem divided>
              ctrl/⌘+z = 撤销
            </DropdownItem>
            <DropdownItem>
              ctrl/⌘+shift+z = 恢复
            </DropdownItem>
            <DropdownItem name="github" style="color:red" divided>觉得好用，去点个star ？</DropdownItem>
          </DropdownMenu>
        </Dropdown>
      </span>
      <span class="center col" span="1">
        <Button @click="makeFile" type="primary">导出</Button>
      </span>
      <span v-show="iconShow" @click="moreCON" class="next iconfont icon-gengduo" size="15"></span>
    </div>

    <Modal v-model="modalShow" title="插入图片" @on-cancel="clear">

      <p> 提示：上传成功后会自动插入到编辑器中，请耐心等待。</p>
      <Upload ref="upload" :data="{ssl: true}" name="iufile" type="drag" accept="image/*" :show-upload-list="true" :on-success="uploadSuccess" :on-error="uploadError" action="https://www.17uw.cn/api/upload/weibo">
        <div style="padding: 20px 0">
          <Icon type="ios-cloud-upload" size="52" style="color: #3399ff"></Icon>
          <p>点击或拖拽上传</p>
        </div>
      </Upload>
      选择清晰度：
      <RadioGroup v-model="uploadType">
        <Radio label="thumbnail">模糊（缩略图）</Radio>
        <Radio label="mw690">一般（中等尺寸）</Radio>
        <Radio label="large">高清（原图）</Radio>
      </RadioGroup>
      <div slot="footer"></div>
    </Modal>
  </div>
</template>
<script>
import bus from "./../lib/bus.js";
export default {
  name: "Header",
  data() {
    return {
      linkText: "", // 链接文字
      linkAddress: "", // 链接地址
      modalShow: false,
      uploadType: "mw690",
      iconShow: false,
      more: false,
      actionBtnList: [
        // 操作按钮
        {
          desc: "加粗",
          template: " **text** ",
          icon: "icon-jiacu",
          bothEnds: "**"
        },
        {
          desc: "倾斜",
          template: " *text* ",
          icon: "icon-qingxie",
          bothEnds: "*"
        },
        {
          desc: "删除线",
          template: " ~~text~~ ",
          icon: "icon-shanchuxian",
          bothEnds: "~~"
        },
        {
          desc: "下划线",
          template: " ++text++ ",
          icon: "icon-Underline",
          bothEnds: "++"
        },
        {
          desc: "行内代码",
          template: " `text` ",
          icon: "icon-kuohaobrackets"
        },
        {
          desc: "插入链接",
          template: "link",
          isSpecial: true, // 特殊标记，当存在特殊标记时，template作为该按钮调用的函数名传入
          icon: "icon-link"
        },
        {
          desc: "插入图片",
          template: "insertImage",
          isSpecial: true, // 特殊标记，当存在特殊标记时，template作为该按钮调用的函数名传入
          icon: "icon-image"
        },
        {
          desc: "标记",
          template: ` ==text== `,
          icon: "icon-biaozhu",
          bothEnds: "=="
        },
        {
          desc: "一级标题",
          template: `
# text`,
          icon: "icon-h1"
        },
        {
          desc: "二级标题",
          template: `
## text`,
          icon: "icon-h2"
        },
        {
          desc: "三级标题",
          template: `
### text`,
          icon: "icon-h3"
        },
        {
          desc: "四级标题",
          template: `
#### text`,
          icon: "icon-h4"
        },
        {
          desc: "五级标题",
          template: `
##### text`,
          icon: "icon-h5"
        },
        {
          desc: "六级标题",
          template: `
###### text`,
          icon: "icon-h6"
        },
        {
          desc: "分割线",
          template: `
***`,
          icon: "icon-fengexian"
        },
        {
          desc: "待办",
          template: `
- [ ] text`,
          icon: "icon-checkboxoutlineblank"
        },
        {
          desc: "已完成",
          template: `
- [x] text`,
          icon: "icon-checkboxoutline"
        },
        {
          desc: "引用",
          template: `
> text`,
          icon: "icon-yinyong"
        },
        {
          desc: "无序列表",
          template: `
- text - 1
- text - 2`,
          icon: "icon-richtextbulletedlist"
        },
        {
          desc: "有序列表",
          template: `
1. text - 1
1. text - 2 `,
          icon: "icon-youxuliebiao"
        },
        {
          desc: "代码块",
          template: `
\`\`\`
code here!
\`\`\``,
          icon: "icon-code"
        },
        {
          desc: "插入表格",
          template: `
1header 1 | header 2
---|---
text1 | text2
text1 | text2`,
          icon: "icon-biaoge"
        }
      ]
    };
  },

  methods: {
    goToGithub(name) {
      if (!name) return;
      window.open("https://github.com/xuzhongpeng/note-md.git");
    },
    uploadError(error) {
      this.$Notice.error({
        title: "上传出错"
      });
      this.clear();
    },
    uploadSuccess(response, file, fileList) {
      if (response.code != "success") {
        this.$Notice.error({
          title: response.msg
        });
        return;
      }
      const pattern = /\w{25,40}/;
      const str = response.data.url;
      const myPid = str.match(pattern)[0];
      // alert(this.makeImageUrl(myPid))
      const finalUrl = this.makeImageUrl(myPid);
      bus.$emit(
        "insert",
        `
![](${finalUrl})`
      );
      this.clear();
    },
    clear() {
      setTimeout(() => {
        this.modalShow = false;
        this.uploadType = "mw690";
        this.$refs.upload.clearFiles();
      }, 250);
    },
    crc32(str) {
      // pic 解码算法
      str = String(str);
      var table =
        "00000000 77073096 EE0E612C 990951BA 076DC419 706AF48F E963A535 9E6495A3 0EDB8832 79DCB8A4 E0D5E91E 97D2D988 09B64C2B 7EB17CBD E7B82D07 90BF1D91 1DB71064 6AB020F2 F3B97148 84BE41DE 1ADAD47D 6DDDE4EB F4D4B551 83D385C7 136C9856 646BA8C0 FD62F97A 8A65C9EC 14015C4F 63066CD9 FA0F3D63 8D080DF5 3B6E20C8 4C69105E D56041E4 A2677172 3C03E4D1 4B04D447 D20D85FD A50AB56B 35B5A8FA 42B2986C DBBBC9D6 ACBCF940 32D86CE3 45DF5C75 DCD60DCF ABD13D59 26D930AC 51DE003A C8D75180 BFD06116 21B4F4B5 56B3C423 CFBA9599 B8BDA50F 2802B89E 5F058808 C60CD9B2 B10BE924 2F6F7C87 58684C11 C1611DAB B6662D3D 76DC4190 01DB7106 98D220BC EFD5102A 71B18589 06B6B51F 9FBFE4A5 E8B8D433 7807C9A2 0F00F934 9609A88E E10E9818 7F6A0DBB 086D3D2D 91646C97 E6635C01 6B6B51F4 1C6C6162 856530D8 F262004E 6C0695ED 1B01A57B 8208F4C1 F50FC457 65B0D9C6 12B7E950 8BBEB8EA FCB9887C 62DD1DDF 15DA2D49 8CD37CF3 FBD44C65 4DB26158 3AB551CE A3BC0074 D4BB30E2 4ADFA541 3DD895D7 A4D1C46D D3D6F4FB 4369E96A 346ED9FC AD678846 DA60B8D0 44042D73 33031DE5 AA0A4C5F DD0D7CC9 5005713C 270241AA BE0B1010 C90C2086 5768B525 206F85B3 B966D409 CE61E49F 5EDEF90E 29D9C998 B0D09822 C7D7A8B4 59B33D17 2EB40D81 B7BD5C3B C0BA6CAD EDB88320 9ABFB3B6 03B6E20C 74B1D29A EAD54739 9DD277AF 04DB2615 73DC1683 E3630B12 94643B84 0D6D6A3E 7A6A5AA8 E40ECF0B 9309FF9D 0A00AE27 7D079EB1 F00F9344 8708A3D2 1E01F268 6906C2FE F762575D 806567CB 196C3671 6E6B06E7 FED41B76 89D32BE0 10DA7A5A 67DD4ACC F9B9DF6F 8EBEEFF9 17B7BE43 60B08ED5 D6D6A3E8 A1D1937E 38D8C2C4 4FDFF252 D1BB67F1 A6BC5767 3FB506DD 48B2364B D80D2BDA AF0A1B4C 36034AF6 41047A60 DF60EFC3 A867DF55 316E8EEF 4669BE79 CB61B38C BC66831A 256FD2A0 5268E236 CC0C7795 BB0B4703 220216B9 5505262F C5BA3BBE B2BD0B28 2BB45A92 5CB36A04 C2D7FFA7 B5D0CF31 2CD99E8B 5BDEAE1D 9B64C2B0 EC63F226 756AA39C 026D930A 9C0906A9 EB0E363F 72076785 05005713 95BF4A82 E2B87A14 7BB12BAE 0CB61B38 92D28E9B E5D5BE0D 7CDCEFB7 0BDBDF21 86D3D2D4 F1D4E242 68DDB3F8 1FDA836E 81BE16CD F6B9265B 6FB077E1 18B74777 88085AE6 FF0F6A70 66063BCA 11010B5C 8F659EFF F862AE69 616BFFD3 166CCF45 A00AE278 D70DD2EE 4E048354 3903B3C2 A7672661 D06016F7 4969474D 3E6E77DB AED16A4A D9D65ADC 40DF0B66 37D83BF0 A9BCAE53 DEBB9EC5 47B2CF7F 30B5FFE9 BDBDF21C CABAC28A 53B39330 24B4A3A6 BAD03605 CDD70693 54DE5729 23D967BF B3667A2E C4614AB8 5D681B02 2A6F2B94 B40BBE37 C30C8EA1 5A05DF1B 2D02EF8D";
      if (typeof crc == "undefined") {
        crc = 0;
      }
      var x = 0,
        y = 0;
      crc = crc ^ -1;
      for (var i = 0, iTop = str.length; i < iTop; i++) {
        y = (crc ^ str.charCodeAt(i)) & 0xff;
        x = "0x" + table.substr(y * 9, 8);
        crc = (crc >>> 8) ^ x;
      }
      return crc ^ -1;
    },
    makeImageUrl(pid) {
      let url = "";
      let zone = "";
      const url_pre = "https://ws";

      if (typeof this.uploadType == "undefined") this.uploadType = "large";

      if (pid[9] == "w") {
        zone = (this.crc32(pid) & 3) + 1;
        url = url_pre + zone + ".sinaimg.cn/" + this.uploadType + "/" + pid;
      } else {
        zone = ((pid.substr(-2, 2), 16) & 0xf) + 1;
        url = url_pre + zone + ".sinaimg.cn/" + this.uploadType + "/" + pid;
      }

      if (pid[21] == "g") {
        return url + ".gif";
      } else {
        return url + ".jpg";
      }
    },
    insert(temp, isSpecial = false, bothEnds = "") {
      if (!isSpecial) bus.$emit("insert", temp, bothEnds);
      else this[temp]();
    },
    link() {
      const _this = this;
      this.$Modal.confirm({
        title: "插入链接",
        render: h => {
          return h("Form", [
            h(
              "FormItem",
              {
                props: {
                  label: "显示文字:"
                }
              },
              [
                h("Input", {
                  props: {
                    type: "text",
                    placeholder: "输入显示文字",
                    maxlength: 10
                  },
                  on: {
                    input: e => {
                      this.linkText = e;
                    }
                  }
                })
              ]
            ),
            h(
              "FormItem",
              {
                props: {
                  label: "链接地址:"
                }
              },
              [
                h("Input", {
                  props: { type: "url", placeholder: "输入链接地址" },
                  on: {
                    input: e => {
                      this.linkAddress = e;
                    }
                  }
                })
              ]
            )
          ]);
        },
        onOk() {
          bus.$emit("insert", `[${_this.linkText}](${_this.linkAddress})`);
        }
      });
    },
    insertImage() {
      this.modalShow = true;
    },
    makeFile() {
      if (!("download" in document.createElement("a")))
        return this.$Notice.error({ title: "浏览器不支持" });
      this.$Modal.confirm({
        title: "插入链接",
        content: "文件导出后将会清除草稿，是否确认",
        onOk() {
          bus.$emit("export");
        }
      });
    },
    setSize() {
      var docWidth = document.body.clientWidth;
      console.log(this);
      if (docWidth <= parseInt(1242)) {
        this.iconShow = true;
      } else {
        this.iconShow = false;
      }
    },
    startMover(dom,position,size,time) {
      //目标值
      var myTime=time/size*5;
      //clearInterval(this.timer); //执行当前动画同时清除之前的动画
      var odiv = dom;
      var position1=position.substring(0,1).toUpperCase()+position.substring(1);
      var myPosition='client'+position1;
      var nowSize=odiv[myPosition];
     // debugger
      var timer =setInterval(function() {
        var speed = 0;
        if (nowSize > size) {
          nowSize -=5;
          if (nowSize <= size) {
            clearInterval(timer);
          }
        } else {
          nowSize += 5;
          if (nowSize >= size) {
            clearInterval(timer);
          }
        }
        odiv.setAttribute('style',position+':'+nowSize + "px");        
      }, myTime);
    },
    moreCON() {
      if (!this.more) {
        this.startMover(document.getElementsByClassName('header')[0],'height',100,500);
        document.getElementsByClassName('g_editor_container')[0].setAttribute('style','height:calc(100% - 104px)')
        document
          .getElementsByClassName("next")[0]
          .setAttribute("style", "transform: rotate(0deg);");
        this.more = true;
      } else {
        this.startMover(document.getElementsByClassName('header')[0],'height',50,300);
        setTimeout(()=>{document.getElementsByClassName('g_editor_container')[0].setAttribute('style','height:calc(100% - 51px)')},400)
        document
          .getElementsByClassName("next")[0]
          .setAttribute("style", "transform: rotate(270deg);");
        this.more = false;
      }
    }
  },
  mounted() {
    this.setSize();
    window.onresize = this.setSize.bind(this);
  }
};
</script>
<style lang="less" scoped>
.header {
  height: 50px;
  line-height: 40px;
  border-bottom: 1px #ccc solid;
  box-shadow: 2px 2px 2px #ccc;
  padding: 5px 10px 0 10px;
  overflow: hidden;
  span {
    display: inline-block;
  }
  .col1 {
    width: 50px;
  }
  .col {
    width: 60px;
  }
  .center {
    text-align: center;
  }
  .next {
    cursor: pointer;
    position: fixed;
    top: 4px;
    right: 12px;
    color: #808080;
    font-size: 12px;
    transform: rotate(270deg);
  }
  .m_action_item {
    cursor: pointer;
    &:hover {
      color: rgb(11, 155, 212);
      font-weight: 900;
    }
  }
}
</style>

