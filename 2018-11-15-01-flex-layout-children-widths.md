# Flex Layout - Children Widths

So far, we have *not* explicitly declared children widths. We assumed whatever width they would come in depending on their content. In this article, we will explore how
to set children's widths.

## The `fxFlex` Directive

We can apply `fxFlex` on child elements in order to specify their widths. Be forewarned that declaring widths is *not* exactly as straight-forward as setting a percentage
value and calling it a day. Many