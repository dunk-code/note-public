[源码](https://gitee.com/jeesys/EasyCaptcha/)

验证码图片接口

```java
@RequestMapping("/images/captcha")
public void captcha(HttpServletRequest request, HttpServletResponse response) {
    CaptchaUtil captcha = new CaptchaUtil();
    try {
        captcha.out(120, 25, 4, request, response);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

校验验证码是够正确（基于Session）

```java
@PostMapping("/login")
@ResponseBody
public ResponseMessage login(@RequestParam(name = "captcha" , required = false)String captcha,
                     @RequestParam(name = "username", required = false)String username,
                     @RequestParam(name = "password", required = false)String password,
                     HttpSession session) {
    String serverCaptcha = (String) session.getAttribute("captcha");
    ResponseMessage response = new ResponseMessage();
    if (!serverCaptcha.equals(captcha)) {
        response.setStatus(false);
        response.setReason("验证码不正确");
    } else {
        if ("dunkcode".equals(username) && "123".equals(password)) {
            response.setStatus(true);
        } else {
            response.setStatus(false);
            response.setReason("用户名或密码不正确");
        }
    }
    return response;
}
```

