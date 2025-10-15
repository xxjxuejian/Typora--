https://web.dev/articles/lcp?hl=zh-cn#what_elements_are_considered



```
import { DirectiveBinding } from "vue";

const lazyLoad = (el: HTMLImageElement, binding: DirectiveBinding<string>) => {
  const loadImage = () => {
    el.src = binding.value;
  };

  if ("IntersectionObserver" in window) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          loadImage();
          observer.unobserve(el);
        }
      });
    });
    // 监听当前指令绑定的元素
    observer.observe(el);
  } else {
    // 不支持IntersectionObserver的浏览器直接加载
    loadImage();
  }
};

export default {
  mounted(el: HTMLImageElement, binding: DirectiveBinding<string>) {
    lazyLoad(el, binding);
  },
};
```
