# cross-domain-iframe-proxy
跨域修改iframe页面内容

## Usage
将proxy.html页面放到iframe页面同域的服务下, 并且可以访问, 主站点通过proxy.html与iframe目标页面交互, 参数格式:
{
    origin,
    iframe,
    includeStyle,
    includeScript,
    importStyle,
    importScript,
}
支持2种传参方式:
postMessage 和 url, 使用url时需要使用encodeURIComponent编码

### Use postMessage
```jsx
function proxy(props) {
    handleLoad = (e) => {
        e.target.contentWindow.postMessage(JSON.stringify({
            iframe: `<iframe name="target" title="target" className="target" src="https://www.target.com/index.html" frameBorder="0" scrolling="no" style="width: 100%;height:100%"></iframe>`,
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
            importStyle: `http://www.a.com/assets/css/import.css`,
            importScript: `http://www.a.com/assets/css/import.js`
        }), 'https://www.target.com');
    }

    return <iframe name="proxy" title="proxy" className="proxy" width="100%" height="100%" onLoad={handleLoad} src={`https://www.target.com/proxy.html?origin=${window.location.protocol}//${window.location.host}`} frameBorder="0" scrolling="no"></iframe>;
}
```

### Use URL
```jsx
function proxy() {
    var options = 'iframe=' + encodeURIComponent(`
        <iframe name="mapiframe" title="mapiframe" className="mapiframe" src="https://www.target.com/index.html" frameBorder="0" scrolling="no" style="width: 100%;height:100%"></iframe>
    `);

    options += '&includeStyle=' + encodeURIComponent(`
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

    options += '&includeScript=' + encodeURIComponent(`
        window.addEventListener('load', function(event) {
            alert(document.querySelector('body').innerHTML);
        });
    `);

    options += '&importStyle=' + encodeURIComponent(`
        http://www.a.com/assets/css/import.css
    `);

    options += '&importScript=' + encodeURIComponent(`
        http://www.a.com/assets/css/import.js
    `);

    return <iframe name="proxy" title="proxy" className="proxy" width="100%" height="100%" src={`https://www.target.com/proxy.html?${options}`} frameBorder="0" scrolling="no"></iframe>;
}

```