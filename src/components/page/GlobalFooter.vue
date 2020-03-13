<template>
  <div class="footer">
    <div class="copyright">
      Copyright
      <a-icon type="copyright"/>
      2020 <span>北京优福安康医护用品有限责任公司</span>
    </div>
    <div class="links">
<!--      <a href="http://www.jeecg.org" target="_blank">JEECG 首页</a>-->
      <a href="http://www.beian.miit.gov.cn" target="_blank"><img src="~@/assets/ghs.png"/>闽ICP备19022290号-1</a>
<!--      <a href="https://github.com/zhangdaiscott/jeecg-boot" target="_blank">-->
<!--        <a-icon type="github"></a-icon>-->
<!--        闽ICP备19022290号-1-->
<!--      </a>-->
    </div>
  </div>
</template>

<script>
  export default {
    name: "LayoutFooter"
  }
</script>

<style lang="scss" scoped>
  .footer {
    padding: 0 16px;
    margin: 48px 0 24px;
    text-align: center;

    .links {
      margin-bottom: 8px;

      a {
        color: rgba(0, 0, 0, .45);

        &:hover {
          color: rgba(0, 0, 0, .65);
        }

        &:not(:last-child) {
          margin-right: 40px;
        }
      }
    }
    .copyright {
      color: rgba(0, 0, 0, .45);
      font-size: 14px;
    }
  }
</style>