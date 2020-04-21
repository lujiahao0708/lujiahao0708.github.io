---
title: emall商城-SSO单点登录
categories: JavaEE
tags:
  - JavaEE
  - 商城
description: 单点登录实现
abbrlink: 9f6a058f
date: 2017-10-18 11:30:05
---

## 1. 前期准备

1. 准备一台Redis服务器
2. 添加host`127.0.0.1 sso.emall.com`
3. 搭建`emall-sso`工程并整合响应的框架

## 2. 实现原理

>单点登录的场景随处可见,此功能极大的简化了用户在网站间的重复登录,使得用户体验更加良好.本教程单点登录的实现原理:用户根据用户名和密码登录,成功后服务器返回token信息,并将token信息写入Redis和Cookie中,用户再次登录时,首先判断Cookie中是否有token信息,如果有则根据token去后台换取用户信息;否则提示用户重新登录.

## 3. 实现步骤

### 3.1 登录接口

	/**
     * 用户登录
     */
    @Override
    public CommonResult userLogin(UserDTO userDTO, HttpServletRequest request, HttpServletResponse response) {
        String username = userDTO.getUsername();
        String password = userDTO.getPassword();

        TbUserExample example = new TbUserExample();
        TbUserExample.Criteria criteria = example.createCriteria();
        criteria.andUsernameEqualTo(username);
        List<TbUser> list = tbUserMapper.selectByExample(example);
        // 如果没有此用户名  没用用户名也返回这个信息是因为防止猜测用户名
        if (list == null || list.size() == 0) {
            return CommonResult.build(400, "用户名或密码错误");
        }
        TbUser tbUser = list.get(0);
        // 对比密码
        if (!DigestUtils.md5DigestAsHex(password.getBytes()).equals(tbUser.getPassword())) {
            return CommonResult.build(400, "用户名或密码错误");
        }
        // 生成token
        String token = UUID.randomUUID().toString();
        // 保存用户信息前先把密码清除,为了安全起见
        tbUser.setPassword(null);

        // 用户信息存入Redis
        saveUserInfoToRedis(tbUser, token);

        // 添加写cookie的逻辑  cookie有效期是关闭浏览器失效
        CookieUtils.setCookie(request, response, COOKIE_TOKEN, token);
        return CommonResult.ok(token);
    }

### 3.2 根据token查询用户信息

	/**
     * 根据token查询用户信息
     */
    @Override
    public CommonResult getUserByToken(String token) {
        // 根据token从redis中查询用户信息
        String json = jedisClientDao.get(REDIS_USER_SESSION_KEY + ":" + token);
        if (StringUtils.isBlank(json)) {
            return CommonResult.build(400, "此Session已经过期,请重新登录");
        }
        // 更新过期时间
        jedisClientDao.expire(REDIS_USER_SESSION_KEY + ":" + token, SSO_SESSION_EXPIRE);
        // 返回用户信息
        return CommonResult.ok(JsonUtils.jsonToPojo(json, TbUser.class));
    }

## 4. 跨域

### 4.1 JSONP

JSONP的实现与 ajax 没有任何关系，JSONP是通过script的src实现的，最终都是向服务器发送请求数据然后回调，而且方便起见，jquery把 JSONP 封装在了 $.ajax 方法中，调用方式与 ajax 调用方式略有区别。JSONP本质是:动态创建script标签，然后通过他的src属性发送跨域请求.

	/**
     * 通过token查询用户信息
     *
     * @param token
     * @return
     */
    @RequestMapping(value = "/token/{token}")
    @ResponseBody
    public Object getUserByToken(@PathVariable String token, String callback) {
        CommonResult result = null;
        try {
            result = userService.getUserByToken(token);
        } catch (Exception e) {
            e.printStackTrace();
            result = CommonResult.build(500, ExceptionUtil.getStackTrace(e));
        }
        // 判断是否为jsonp调用
        if (StringUtils.isBlank(callback)) {
            // 不是jsonp调用
            return result;
        } else {
            MappingJacksonValue mappingJacksonValue = new MappingJacksonValue(result);
            mappingJacksonValue.setJsonpFunction(callback);
            return mappingJacksonValue;
        }
    }

### 4.2 CORS

	此种方式后端实现有两种方式:
		让所有的controller类继承自定义的BaseController类，改类中将对返回的头部做些特殊处理;
		通过filter实现所有的请求封装跨域.

#### 4.2.1 继承BaseController

	public abstract class BaseController {
	  	/**
	     * description:send the ajax response back to the client side
	     * @param responseObj
	     * @param response
	     */
	    protected void writeAjaxJSONResponse(Object responseObj, HttpServletResponse response) {
	        response.setCharacterEncoding("UTF-8");
	        response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate"); // HTTP 1.1
	        response.setHeader("Pragma", "no-cache"); // HTTP 1.0
	        /**
	         * for ajax-cross-domain request TODO get the ip address from
	         * configration(ajax-cross-domain.properties)
	         */
	        response.setHeader("Access-Control-Allow-Origin", "*");
	        response.setDateHeader("Expires", 0); // Proxies.
	        PrintWriter writer = getWriter(response);
	        writeAjaxJSONResponse(responseObj, writer);
	    }

	  	/**
	     *
	     * @param response
	     * @return
	     */
	    protected PrintWriter getWriter(HttpServletResponse response) {
	        if(null == response){
	            return null;
	        }
	        PrintWriter writer = null;
	        try {
	            writer = response.getWriter();
	        } catch (IOException e) {
	            logger.error("unknow exception", e);
	        }
	        return writer;
	    }

	    /**
	     * description:send the ajax response back to the client side.
	     *
	     * @param responseObj
	     * @param writer
	     * @param writer
	     */
	    protected void writeAjaxJSONResponse(Object responseObj, PrintWriter writer) {
	        if (writer == null || responseObj == null) {
	            return;
	        }
	        try {         
	        	writer.write(JSON.toJSONString(responseObj,SerializerFeature.DisableCircularReferenceDetect));
	        } finally {
	            writer.flush();
	            writer.close();
	        }
	    }
	}

#### 4.2.2 Filter实现

	/**
	 * @author lujiahao
	 * @version 1.0
	 * @date 2017-10-15 22:20
	 */
	public class HeadersCORSFilter implements Filter {
	    @Override
	    public void init(FilterConfig filterConfig) throws ServletException {

	    }

	    @Override
	    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	        HttpServletResponse response = (HttpServletResponse) servletResponse;
	        response.setHeader("Access-Control-Allow-Origin","*");
	        filterChain.doFilter(servletRequest, response);
	    }

	    @Override
	    public void destroy() {

	    }
	}

web.xml中配置:

	<!-- Ajax Access-Control-Allow-Origin 跨域 拦截器解决方案 -->
    <filter>
        <filter-name>ACAOFilter</filter-name>
        <filter-class>com.lujiahao.sso.filter.HeadersCORSFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>ACAOFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

## 5. 代码详见[emall-sso](https://github.com/chiahaolu/emall/tree/master/emall-sso)

## 6. 实现效果

![](https://github.com/chiahaolu/emall/blob/master/gif/emall-sso.gif)

## 7. 说明

emall商城系列是整合[淘淘商城]和[慕课网Java从零到企业级电商项目实战]的系列,这两部教程来自互联网.