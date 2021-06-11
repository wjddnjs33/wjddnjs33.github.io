---
layout: post
date: 2021-01-20 00:00:00
title: "0-day RCE in vBulletin"
categories: [Analysis, CVE]
tags: [0-day, RCE]
author:
  - Jeongwon Jo
---
vBulletin에서 발견된 0-day 취약점을 분석해보겠습니다. vBulletin은 외국에서 사용하는 포럼 툴이라고 합니다. 한국 말로 풀어 쓰면 토론형 게시판이라고 볼 수 있겠습니다. 여하튼 이 포럼 툴에서 RCE 취약점이 총 2번 발견 되었습니다.

|CVE ID|Version|CVSS|
|----|----|----|
|CVE-2019-16759|5.x ~ 5.5.4|9.8|
|CVE-2020-7373|5.5.4 ~ 5.6.2|9.8|

처음에는 `CVE-2019-16759`라는 취약점이 나왔었고, 취약한 버전은 5.x부터 5.5.4까지라고 합니다. 해당 취약점은 9월 24일에 처음 POC가 공개되었고, 바로 다음 날인 25일에 패치가 됐다고 합니다.

하지만 완전히 패치된 줄 알았던 0-day 취약점이 한 보안 연구원에 의해서 또 다시 뚫리고 말았습니다. `CVE-2020-7373`라는 CVE ID로 나왔었고, 취약한 버전은 5.5.4부터 5.6.2까지라고 합니다. 그럼 한 번 RCE가 어떻게 트리거 됐는지 코드 분석을 통해 알아보겠습니다.

---
## CVE-2019-16759

```php
// index.php

(생략)

//For a few set routes we can run a streamlined funcction.
if (vB5_Frontend_ApplicationLight::isQuickRoute())
{
    $app = vB5_Frontend_ApplicationLight::init('config.php');
    if ($app->execute())
    {
        exit();
    }
}

(생략)
```
취약점의 시작은 index.php에서 시작이 됩니다. `isQuickRoute()`의 값이 참이면 if 문 내부로 들어가는 것을 볼 수 있습니다. isQuickRoute 메서드는 `includes/vb5/frontend/applicationlight.php` 내부에 정의되어 있습니다.

```php
// includes/vb5/frontend/applicationlight.php

(생략)

  /** Tells whether this class can process this request
   *
   * @return boll
   */
  public static function isQuickRoute()
  {
      if (empty($_REQUEST['routestring']))
      {
          return false;
      }
      
      if (isset(self::$quickRoutes[$_REQUEST['routestring']]))
      {
          return true;
      }
      
      if (substr($_REQUEST['routestring'], 0, 8) == 'ajax/api')
      {
          return true;
      }
      
      if (substr($_REQUEST['routestring'], 0, 11) == 'ajax/render')
      {
          return true;
      }
      
      return false;
  }


(생략)

```
`includes/vb5/frontend/applicationlight.php` 내부에는 위와 같이 isQuickRoute 메서드가 정의되어 있는데 routestring의 값이 `'ajax/api'`, `'ajax/render'`이면 ture를 반환해 vB5_Frontend_ApplicationLight::init가 실행이 됩니다.

```php
// includes/vb5/frontend/applicationlight.php

(생략)

    /**Standard constructor. We only access application throuth init() **/
    protected function __construct()
    {
        if (empty($_REQUEST['routestring]))
        {
            return false;
        }
        
        if (isset(self::$quickRoutes[$_REQUEST['routestring']]))
        {
            $this->application = self::$quickRoutes[$_REQUEST['routestring]];
            return true;
        }
        
        if (substr($_REQUEST['routestring], 0, 8) == 'ajax/api')
        {
            $this->application = array('handler' => 'handleAjaxApi', 'static' => false);
            
            if (substr($_REQUEST['routestring'], 0, 17) == 'ajax/api/cron/run')
            {
                $this->application['runcron'] = true;
            }
            return true;
        }
        
        if (substr($_REQUEST['routestring'], 0, 11) == 'ajax/render')
        {
            $this->application = array('handler' => 'callRender', 'static' => false);
            return true;
        }
        
        return false;  
    }    

(생략)
```
index.php에서 vB5_Frontend_ApplicationLight::init 메서드가 실행되면 위 `__construct` 메서드가 실행됩니다. 코드 하단에 보면 `routestring`의 값이 `'ajax/render'`이면 handler로 callRender 메서드가 설정되어 있는 것을 볼 수 있습니다. callRender 메서드도 `includes/vb5/frontend/applicationlight.php` 내부에 정의되어 있습니다.

```php
// includes/vb5/frontend/applicationlight.php

(생략)

    /** This renders a template from an ajax call
    */
    protected function callRender()
    {
        $routeInfo = explode('/', $_REQUEST['routestring']);
        
        if (count($routeInfo) < 3)
        {
            throw new vB5_Exception_Api('ajax', 'api', array(), 'invalid_request');
        }
        
        $params = array_merge($_POST, $_GET);
        $this->router = new vB5_Frontend_Routing();
        $this->router->setRouteInfo(array('action' => 'actionRender', 'arguments' => $params,
            'template' => $routeInfo[2], 'queryParameters' => $_GET));
        Api_InterfaceAbstract::setLight();
        $this->sendAsJson(vB5_Template::staticRenderAjax($routeInfo[2], $params));
    }    

(생략)
```
callRender 메서드를 보면 $routeInfo에 explode() 메서드를 이용해서 `$_REQUEST['routestring']`의 값을 슬래쉬를 기준으로 잘라 배열로 만들어주고, $params에는 array_merge 메서드를 이용해서 Query String 값과 Raw Data 값을 합쳐 하나의 배열로 만드는 것을 볼 수 있습니다.
그리고 $routeInfo[2]의 값과 $params의 값을 템플릿으로 전송하는 것을 볼 수 있습니다.

```xml
// core/install/vbulletin-style.xml

(생략)

        <template name="widget_php" templatetype="template" date="1372889589" username="vBulletin Solutions" version="5.0.5 Release Canddate 1"><![CDATA[<
        vb:if condition="empty(widgetConfig) AND !empty($widgetinstaceid)">
    {vb:data widgetConfig, widget, fetchConfig, (vb:raw widgetinstanceid)}
</vb:if>
<vb:if condition="!empty($widgetConfig)">
    {vb:set widgetid, {vb:raw widgetConfig.widgetid}}
    {vb:set widgetinstanceid, {vb:raw widgetConfig.widgetinstanceid}}
</vb:if>

<div class="canvax-widget default-widget custtom-html-widget" data-widget-id="{vb:raw widgetid}" data-widget-instance-id="{vb:raw widgetinstanceid}">
    {vb:template module_title, widgetConfig={vb:raw widgetConfig}, can_use_sitebuilder={vb:raw user.can_use_sitebuilder}}
    <div class="widget-header-divder" />
        <hr class="widget-header-divider" />
        <vb:if condition="!empty($widgetConfig['code']) AND !$vboptions['disable_php_rendering']">
            {vb:action evaledPHP, bbcode, evalCode, {vb:raw widgetConfig.code}}
            {vb:raw $evaledPHP}
        <vb:else />
            <vb:if condition="$user[can_use_sitebuilder']">
                <span class="note">{vb:phrase click_edit_to_config_module}</span>
            </vb:if>
        </vb:if>
    </div>
<div>]]></template>

(생략)
```
widget_php 템플릿을 확인해보면 `$widgetConfig['code']`의 값이 비어있지 않고, `!$vboptions['disable_php_rendering']`가 비활성화 되어 있으면 다음 코드를 코드를 실행하는 것을 볼 수 있습니다.

```xml
{vb:action evaledPHP, bbcode, evalCode, {vb:raw widgetConfig.code}}
{vb:raw $evaledPHP}
```

```php
// includes/vb5/frontend/controller/bbcode.php

(생략)
    function evalCode($code)
    {
        ob_start();
        eval($code);
        $output = ob_get_contents();
        ob_end_clean();
        return $output;
    }

(생략)
```
evalCode 메서드는 `includes/vb5/frontend/controller/bbcode.php` 내에 정의하고 있고, eval 함수를 이용해서 $code 값을 실행시키고, `$output` 값을 리턴해주는 것을 볼 수 있습니다.

결론적으로 RCE를 트리거 하기 위해서는 `includes/vb5/frontend/applicationlight.php`에서 `isQuickRoute` 메서드에서 rotuestring의 값이 'ajax/render'여야 하고, callRender에서 $routeInfo[2]의 값은 `'widget_php'`여야 하고, `widget_php` 템플릿에서는 `$widgetConfig['code']`의 값이 존재해야 하고, eval 함수의 인자 값으로도 `$widgetConfig['code']`가 들어갑니다.

```php
$routeInfo = explode('/', $_REQUEST['routestring']);
$params = array_merge($_POST, $_GET);
print_r($routeInfo);
print_r($params);

input  : http://141.164.52.207/?routestring=ajax/render/widget_php&widgetConfig['code']=phpinfo();
output : Array ( [0] => ajax [1] => render [2] => widget_php ) Array ( [routestring] => ajax/render/widget_php [widgetConfig] => Array ( ['code'] => phpinfo(); ) )
```
그리고 사용자 값을 받을 때, `$_REQUEST`를 이용해서 받아 오기 때문에 GET, POST 메서드를 이용해서 공격을 수행할 수 있습니다. 그러니 GET으로 익스를 하려면 위와 같이 그냥 값을 넘겨 주게 되면 `substr($_REQUEST['routestring'], 0, 11)`의 값이 `'ajax/render'`이기 때문에 참이 반환되는 동시에 `vB5_Frontend_ApplicationLight::init`가 실행 돼 `__construct` 메서드가 실행이 될 것이고, `__construct` 메서드에서도 `substr($_REQUEST['routestring'], 0, 11)`의 값이 `'ajax/render'`이기 때문에 handler를 callRender로 설정하게 됩니다. callRender 메서드에서는 routeInfo[2]의 값으로는 `'widget_php'`, widgetConfig['code']의 값으로는 `'phpinfo();'`가 들어가 있어 2개의  `widget_php` 템플릿으로 전송이 됩니다. `widget_php`에서는 widgetConfig['code']가 비어있지 않기 때문에 evalCode로 widgetConfig['code']가 전송이 되어 phpinfo(); 함수가 실행이 되며 RCE가 성공적으로 트리거 되게 됩니다.

---
## CVE-2019-16759 Patch

CVE-2019-16759 취약점은 총 2개의 패치가 이루어졌다고 합니다.

```php
         /**
          *      Remove any problematic values from the template
          *      variable arrays before rendering
          */
         //for now don't pass the values through.  These arrays are potentially large
         //and we don't want to make unnecesary copies.  The alternative is to pass by
         //reference which causes it's own headaches.  It's an internal function and the
         //relevant arrays are all class variables.
         private function cleanRegistered()
         {   
                 $disallowedNames = array('widgetConfig');
                 foreach($disallowedNames AS $name)
                 {   
                         unset($this->registered[$name]);
                         unset(self::$globalRegistered[$name]);
                 }
         }
```
첫  패치는 `clearRegistered`라는 함수를 이용해서 widgetConfig라는 값이 사용되면 `unset()` 함수를 이용해서 제거하고 있습니다. 그러니 Query String 이나 Raw Data로 `widgetConfig['code']=phpinfo();`라는 값이 들어오게 되면 widgetConfig가 존재하기 때문에 `unset()`에 의해서 제거가 될 것 입니다.

```php
diff -ur vBulletin/vBulletin/vb5_connect/vBulletin-5.5.4_Patch_Level_2/upload/includes/vb5/frontend/applicationlight.php vBulletin/vBulletin/vb5_connect/vBulletin-5.5.5/upload/includes/vb5/frontend/applicationlight.php
--- vBulletin/vBulletin/vb5_connect/vBulletin-5.5.4_Patch_Level_2/upload/includes/vb5/frontend/applicationlight.php	2020-08-08 06:40:31.356918994 -0500
+++ vBulletin/vBulletin/vb5_connect/vBulletin-5.5.5/upload/includes/vb5/frontend/applicationlight.php	2020-08-08 06:40:40.577517014 -0500
@@ -286,20 +286,32 @@
 			throw new vB5_Exception_Api('ajax', 'render', array(), 'invalid_request');
 		}
 
-		$this->router = new vB5_Frontend_Routing();
-		$this->router->setRouteInfo(array(
-			'action'          => 'actionRender',
-			'arguments'       => $serverData,
-			'template'        => $routeInfo[2],
-			// this use of $_GET appears to be fine,
-			// since it's setting the route query params
-			// not sending the data to the template
-			// render
-			'queryParameters' => $_GET,
-		));
-		Api_InterfaceAbstract::setLight();
+		$templateName = $routeInfo[2];
+		if ($templateName == 'widget_php')
+		{
+			$result = array(
+				'template' => '',
+				'css_links' => array(),
+			);
+		}
+		else
+		{
+			$this->router = new vB5_Frontend_Routing();
+			$this->router->setRouteInfo(array(
+				'action'          => 'actionRender',
+				'arguments'       => $serverData,
+				'template'        => $templateName,
+				// this use of $_GET appears to be fine,
+				// since it's setting the route query params
+				// not sending the data to the template
+				// render
+				'queryParameters' => $_GET,
+			));
+			Api_InterfaceAbstract::setLight();
+			$result = vB5_Template::staticRenderAjax($templateName, $serverData);
+		}
 
-		$this->sendAsJson(vB5_Template::staticRenderAjax($routeInfo[2], $serverData));
+		$this->sendAsJson($result);
 	}
 
 	/**
```
그리고 두 번째 패치는 `$routeInfo[2]`(Template Name)의 값으로 `widget_php`가 들어오게 되면 빈 템플릿과 CSS를 반환하도록 하는 조건문을 추가함으로 0-day에 대응하게 되었습니다.

```php
 	diff -ur vBulletin/vBulletin/vb5_connect/vBulletin-5.5.4_Patch_Level_2/upload/includes/vb5/template/runtime.php vBulletin/vBulletin/vb5_connect/vBulletin-5.5.5/upload/includes/vb5/template/runtime.php
--- vBulletin/vBulletin/vb5_connect/vBulletin-5.5.4_Patch_Level_2/upload/includes/vb5/template/runtime.php	2020-08-08 06:40:31.276913797 -0500
+++ vBulletin/vBulletin/vb5_connect/vBulletin-5.5.5/upload/includes/vb5/template/runtime.php	2020-08-08 06:40:40.493511575 -0500
@@ -12,6 +12,26 @@
 
 class vB5_Template_Runtime
 {
+	//This is intended to allow the runtime to know that template it is rendering.
+	//It's ugly and shouldn't be used lightly, but making some features widely
+	//available to all templates is uglier.
+	private static $templates = array();
+
+	public static function startTemplate($template)
+	{
+		array_push(self::$templates, $template);
+	}
+
+	public static function endTemplate()
+	{
+		array_pop(self::$templates);
+	}
+
+	private static function currentTemplate()
+	{
+		return end(self::$templates);
+	}
+
 	public static $units = array(
 		'%',
 		'px',
@@ -1944,6 +1964,21 @@
 			return '<div style="border:1px solid red;padding:10px;margin:10px;">' . htmlspecialchars($timerName) . ': ' . $elapsed . '</div>';
 		}
 	}
+
+	public static function evalPhp($code)
+	{
+		//only allow the PHP widget template to do this.  This prevents a malicious user
+		//from hacking something into a different template.
+		if (self::currentTemplate() != 'widget_php')
+		{
+			return '';
+		}
+		ob_start();
+		eval($code);
+		$output = ob_get_contents();
+		ob_end_clean();
+		return $output;
+	}
 }
```
또한 evalPhP 메서드의 현재 템플릿이 widget_php가 아니면 Null을 반환하도록 하는 조건문을 추가해서, 다른 템플릿에서 악용될 요소도 제거하는 것을 볼 수 있습니다. (Line 342 ~ 345)

이와 같이 총 2곳을 패치해서 0-day 취약점을 제거했지만 약 1년 정도가 지나고 한 보안 연구원에 의해서 우회가 돼 0-day가 다시 나오게 되었습니다.

---
## *CVE-2020-7373*

`CVE-2020-7373`은 `CVE-2019-16759`를 패치한 것을 우회해 다시 RCE 취약점을 트리거 한 0-day 입니다. 해당 취약점은 본례의 익스 방식과 크게 다를 게 없지만 다르다면 템플릿을 `widget_php`가 아닌 `widget_tabbedcontainer_tab_panel`이라는 템플릿을 이용해서 익스를 진행하였습니다.

```xml
<template name="widget_tabbedcontainer_tab_panel" templatetype="template" date="1532130449" username="vBulletin" version="5.4.4 Alpha 2"><![CDATA[{vb:set panel_id, {vb:concat {vb:var id_prefix}, {vb:var tab_num}}}
<div id="{vb:var panel_id}" class="h-clearfix js-show-on-tabs-create h-hide">
	<vb:comment>
	- {vb:var panel_id} 
	<vb:each from="subWidgets" value="subWidget">
	&nbsp;&nbsp;-- {vb:raw subWidget.template}
	</vb:each>
	</vb:comment>
 
	<vb:each from="subWidgets" value="subWidget">
		{vb:template {vb:raw subWidget.template}, 
			widgetConfig={vb:raw subWidget.config}, 
			widgetinstanceid={vb:raw subWidget.widgetinstanceid},
			widgettitle={vb:raw subWidget.title}, 
			tabbedContainerSubModules={vb:raw subWidget.tabbedContainerSubModules},
			product={vb:raw subWidget.product}
		}
	</vb:each> 
</div>]]></template>
```

`widget_tabbedcontainer_tab_panel` 템플릿은 자식 템플릿을 생성해주는 템플릿이라고 합니다. 하단의 보면 subWidget.template를 이용해서 템플릿 이름을 정하고, {vb:raw subWidget.config}를 이용해서 widgetConfig에 값을 넣어주는 것을 볼 수 있습니다.

```
POC : http://141.164.52.207/?subWidgets[0][template]=widget_php&subWidgets[0][config][code]=phpinfo();
```
그러니 위와 같이 subWidget.template의 값으로 `widget_php`, subWidget.config의 값으로 `code:phpinfo();`로 넘겨주면 template의 값은 `widget_php`, widgetConfig.code의 값은 `phpinfo();`가 될 것 이고, widget_php 템플릿이 생성되면 widgetConfig['code']의 값은 `phpinfo();`이므로 `widget_php`에서 `CVE-2019-16759`와 동일하게 RCE가 트리거 되게 됩니다.

---
## CVE-2020-7373 Patch

<strong>패치 코드가 안 보임</strong><br>

---
