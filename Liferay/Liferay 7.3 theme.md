# Liferay 7.3 theme

## Install liferay bundle

```
cd ~/liferay
mkdir bundles
cd bundles
wget https://github.com/liferay/liferay-portal/releases/download/7.3.7-ga8/liferay-ce-portal-tomcat-7.3.7-ga8-20210610183559721.tar.gz
tar -xvf liferay-ce-portal-tomcat-7.3.7-ga8-20210610183559721.tar.gz
echo "include-and-override=portal-developer.properties" > liferay-ce-portal-7.3.7-ga8/portal-ext.properties
```

## Install dependencies

1. Install `Node v10` from [https://nodejs.org/download/release/v10.24.1/](https://nodejs.org/download/release/v10.24.1/)

1. Run `npm install -g yo@3.x.x gulp` to install the Yeoman and Gulp dependency.

1. Run `npm install -g generator-liferay-theme@9.x.x` to install the Liferay Theme Generator.

## Start Liferay-Tomcat Bundle

```
cd liferay-ce-portal-7.3.7-ga8/tomcat-9.0.43/bin
./catalina.sh run
```

## Create and deploy new theme

```
cd ~/liferay
yo liferay-theme
cd NAME-THEME
npm run deploy
```

For create new theme base on `classic-theme` use this:

```
yo liferay-theme:classic
```

### Menu not working in mobile

Edit `navigation.ftl`:

```
<#if has_navigation && is_setup_complete>
	<button aria-controls="navigationCollapse" aria-expanded="false" aria-label="Toggle navigation" class="navbar-toggler navbar-toggler-right" data-target="#navigationCollapse" data-toggle="liferay-collapse" type="button">
		<span class="navbar-toggler-icon"></span>
	</button>

	<div class="collapse navbar-collapse" id="navigationCollapse">
		<@liferay.navigation_menu default_preferences="${preferences}" />
	</div>
</#if>
```