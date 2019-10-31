# cross-domain-iframe-proxy
跨域修改iframe页面内容

## Precondition
前提条件需要将proxy.html放到与内嵌的iframe页同域的服务下, 并且可以被访问.
主站点通过proxy.html与iframe目标页交互.
![](http://git.bbdops.com/frontend/mazarine-ui/raw/master/resources/images/thinLine.png)

## Usage
支持2种调用方式: 使用 postMessage 和 URL传参.

### Use postMessage
使用 window.postMessage 的方式需要使用JSON.stringify将对象转为字符串.
```jsx
function IframeProxy(props) {
    handleLoad = (e) => {
        e.target.contentWindow.postMessage(JSON.stringify({
            iframe: `<iframe name="target" title="target" className="target" src="http://www.targetdomain.com/target.html" frameBorder="0" scrolling="no" style="width: 100%;height:100%"></iframe>`,
            includeStyle: `
                body {
                    background-color: yellow;
                }
                header {
                    display: none;
                }
                footer {
                    display: none;
                }
            `,
            includeScript: `
                window.addEventListener('load', function() {
                    alert(document.querySelector('body').innerHTML);
                });
            `,
            importStyle: `http://www.mydomain.com/assets/css/import.css`,
            importScript: `http://www.mydomain.com/assets/js/import.js`
        }), 'https://www.target.com');
    }

    return <iframe name="proxy" title="proxy" className="proxy" width="100%" height="100%" onLoad={handleLoad} src={`http://www.targetdomain.com/proxy.html?origin=${window.location.protocol}//${window.location.host}`} frameBorder="0" scrolling="no"></iframe>;
}
```

### Use URL params
使用 URL 传参的方式需要将内容用 encodeURIComponent 编码.
```jsx
function IframeProxy(props) {
    var params = 'iframe=' + encodeURIComponent(`
        <iframe name="target" title="target" className="target" src="http://www.targetdomain.com/target.html" frameBorder="0" scrolling="no" style="width: 100%;height:100%"></iframe>
    `);

    params += '&includeStyle=' + encodeURIComponent(`
        body {
            background-color: red;
        }
        header {
            display: none;
        }
        footer {
            display: none;
        }
    `);

    params += '&includeScript=' + encodeURIComponent(`
        window.addEventListener('load', function(event) {
            alert(document.querySelector('body').innerHTML);
        });
    `);

    params += '&importStyle=' + encodeURIComponent(`
        http://www.mydomain.com/assets/css/import.css
    `);

    params += '&importScript=' + encodeURIComponent(`
        http://www.mydomain.com/assets/js/import.js
    `);

    return <iframe name="proxy" title="proxy" className="proxy" width="100%" height="100%" src={`http://www.targetdomain.com/proxy.html?${params}`} frameBorder="0" scrolling="no"></iframe>;
}
```

## API
```
<iframe src="http://www.targetdomain.com/proxy.html?params"></iframe>;

params: {
    origin: 当前站点的域名, 使用postMessage方式时必填, proxy用来校验发出消息的源域名.
    iframe: 需要内嵌的iframe标签字符串,
    includeStyle: 希望添加到iframe页的css内容,
    includeScript: 希望添加到iframe页的js内容,
    importStyle: 希望引入到iframe页的css资源链接, 如果目标站点使用安全协议(https), 资源链接使用非安全协议(http), 该功能会被浏览器禁止.
    importScript: 希望引入到iframe页的js资源链接, 如果目标站点使用安全协议(https), 资源链接使用非安全协议(http), 该功能会被浏览器禁止.
}
```
注意: 处于安全问题, 默认禁用了 includeScript 和 importScript 功能, 如需启用在proxy.html中将变量 ENABLED_JS_INCLUDE 设置为 true 即可.