## 前端开发常用的工具函数

```javascript
var $ = require( 'jquery' ) ;
    $.extend( $.fn , {
        //jquery serializeJSON
        serializeJSON: function(){
            var serializeObj={};
            var array=this.serializeArray();
            var str=this.serialize();
            $(array).each(function(){
                if(serializeObj[this.name]){
                    if($.isArray(serializeObj[this.name])){
                        serializeObj[this.name].push(this.value);
                    }else{
                        serializeObj[this.name]=[serializeObj[this.name],this.value];
                    }
                }else{
                    serializeObj[this.name]=this.value;
                }
            });
            return serializeObj;
        } ,
        // 自动填充表单
        autoFillData: function( data ){
            for( var name in data  ){
                var $ele = this.find("[name='" + name + "']" ) ,
                    value = data[name] ,
                    type = $ele.attr("type") ;
                switch( type ) {
                    case "radio":
                        $ele.each( function(){
                            var $this = $(this) ,
                                val = $this.val() ;
                            if( value == val ){ $this.prop("checked",true); return false; }
                        } ) ;
                        break;
                    default:
                        $ele.val( value ) ;
                        break;
                }
            }
            return this ;
        } ,
        /* fixedBoard 固定面板 */
        fixedBoard: function( config ){
            var $window = $(window) ;
            var fixedBoard = {
                fixedObj: {},
                isIE6: window.VBArray && !window.XMLHttpRequest,
                fixed: function( $element , config ) {
                    var fixedObj = this.fixedObj,
                        $ele = $element,
                        id = $ele.attr("id") || "fixed_id",
                        csstext = "",
                        c = $ele.height(),
                        isFunction =  config && config.scroll && $.isFunction( config.scroll ) ,
                        scrollFunction = isFunction ? $.proxy( config.scroll , $ele ) : $.noop ;
                    fixedObj[id + "_top"] = $ele.offset().top ;
                    fixedObj[id + "_fun"] = function() {
                        var top = $window.scrollTop() ;
                        if (top > fixedObj[id + "_top"]) {
                            scrollFunction( top ) ;
                            if ($ele.hasClass("fixed")) {
                                return;
                            }
                            $ele.addClass("fixed");
                            if (fixedObj.isIE6) {
                                $ele.css({
                                    position: "absolute"
                                });
                            } else {
                                $ele.css({
                                    position: "fixed"
                                });
                            }
                        } else {
                            $ele.removeClass("fixed");
                            $ele[0].style.position = "";
                        }
                    };
                    if( fixedObj.isIE6 ) {
                        csstext = $ele[0].style.cssText;
                        $ele[0].style.cssText = csstext + ";_top:expression((document).documentElement.scrollTop);";
                    }
                    $window.on("scroll", fixedObj[id + "_fun"]);
                }
            };
            return this.each(function(){
                fixedBoard.fixed( $(this) , config ) ;
            });
        }
    } ) ;
    var tool = {
        // 获取url参数 返回一个对象或者对象的属性值
        getUrlParams: function( name ){
            var url = location.href,
                value = null,
                index = url.indexOf("?");
            if (index < 0) {
                if( name ){
                    return undefined ;
                }else{
                    return {} ;
                }
            } else {
                var paramstr = url.substring(index + 1),
                    obj = {},
                    arrParams = paramstr.split("&");
                for (var i = 0; i < arrParams.length; i++) {
                    var tmp = arrParams[i].split("=");
                    obj[tmp[0]] = tmp[1] ? decodeURIComponent( tmp[1] ) : undefined ;
                }
                if (name) {
                    return obj[name]
                } else {
                    return obj
                }
            }
        } ,
        // 获取url的节点树
        getUrlTree: function(index) {
            var urlNode = location.href.replace(/http:\/\//i, "").replace(/\?.*/i, "").split("/"),
                length = urlNode.length;
            if (index == undefined) {
                return urlNode
            } else {
                return index < 0 ? urlNode[length + index] : urlNode[index]
            }
        } ,
        // 根据url追加参数
        appendParams: function( url , param ){
            var _tmpl = '' ,
                _type = $.type( param ) ,
                _param = _type == 'string' ? param : _type == 'object' ? $.param( param ) : $.trim( param ) ;   // string,object , 其他类型,调用$.trim()
            if( url.indexOf( '?' ) < 0 ){
                _tmpl = url + '?' + _param ;
            }else{
                var length = url.length ;
                if( url.charAt( length - 1 ) == '&' ){
                    _tmpl = url + _param ;
                }else{
                    _tmpl = url + '&' + _param ;
                }
            }
            return _tmpl ;
        } ,
        // cookie 获取，设置
        cookie: function(name, value, options) {
            if (typeof value != 'undefined') {
                // 有value值, 设置cookie
                options = options || {};
                if (value === null) {
                    value = '';
                    options.expires = -1;
                }
                var expires = '';
                if (options.expires && (typeof options.expires == 'number' || options.expires.toUTCString)) {
                    var date;
                    if (typeof options.expires == 'number') {
                        //options.expires以小时为单位
                        date = new Date();
                        date.setTime(date.getTime() + (options.expires * 60 * 60 * 1000));
                    } else {
                        date = options.expires;
                    }
                    expires = '; expires=' + date.toUTCString();
                }
                var path = options.path ? '; path=' + options.path : '; path=/';
                var domain = options.domain ? '; domain=' + options.domain : '';
                var secure = options.secure ? '; secure' : '';
                document.cookie = [name, '=', encodeURIComponent(value), expires, path, domain, secure].join('');
            } else {
                // 只有name值, 获取cookie
                var cookieValue = null;
                if (document.cookie && document.cookie != '') {
                    var cookies = document.cookie.split(';');
                    for (var i = 0; i < cookies.length; i++) {
                        var cookie = jQuery.trim(cookies[i]);
                        if (cookie.substring(0, name.length + 1) == (name + '=')) {
                            cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                            break;
                        }
                    }
                }
                return cookieValue;
            }
        } ,
        // 常用校验
        check: {
            isPhone: function() {
                var ua = navigator.userAgent.toLowerCase(),
                    reg = /iPhone|iPad|Android|ucweb|windows\s+mobile|Windows\s+Phone/i;
                return reg.test(ua)
            },
            isIE6: window.VBArray && !window.XMLHttpRequest,
            isNick: function(str) {
                var nickReg = /^[\u4e00-\u9fa5A-Za-z0-9-_]+$/;
                return nickReg.test(str)
            },
            isEmail: function(str) {
                var emailReg = /^[a-z0-9][\w\.]*@([a-z0-9][a-z0-9-]*\.)+[a-z]{2,5}$/i;
                return emailReg.test(str)
            },
            isMobile: function(str) {
                var mobileReg = /^1[345678][0-9]{9}$/;
                return mobileReg.test(str)
            },
            isTelephone: function(str) {
                var phoneReg = /^0\d{2,3}-\d{5,9}$/;
                return phoneReg.test(str)
            },
            isUrl: function(str) {
                var urlReg = /^http:\/\/([\w-]+\.)+[\w-]+(\/[\w-.\/?%&=]*)?$/;
                return urlReg.test(str)
            },
            isNum: function(str) {
                var numReg = /^[0-9]\d*$/;
                return numReg.test(str)
            },
            isFloatNum: function(str) {
                var floatReg = /^(-?\d+)(\.\d+)?$/;
                return floatReg.test(str)
            },
            isDate: function(str) {
                var dateReg = /^((((1[6-9]|[2-9]\d)\d{2})-(0?[13578]|1[02])-(0?[1-9]|[12]\d|3[01]))|(((1[6-9]|[2-9]\d)\d{2})-(0?[13456789]|1[012])-(0?[1-9]|[12]\d|30))|(((1[6-9]|[2-9]\d)\d{2})-0?2-(0?[1-9]|1\d|2[0-8]))|(((1[6-9]|[2-9]\d)(0[48]|[2468][048]|[13579][26])|((16|[2468][048]|[3579][26])00))-0?2-29-))$/;
                return dateReg.test(str)
            },
            isAnimate: function(style) {
                var prefix = ["webkit", "Moz", "ms", "o"],
                    i, humpString = [],
                    htmlStyle = document.documentElement.style,
                    _toHumb = function(string) {
                        return string.replace(/-(\w)/g, function($0, $1) {
                            return $1.toUpperCase()
                        })
                    };
                for (i in prefix) {
                    humpString.push(_toHumb(prefix[i] + "-" + style))
                }
                humpString.push(_toHumb(style));
                for (i in humpString) {
                    if (humpString[i] in htmlStyle) {
                        return true
                    }
                }
                return false
            }
        }
    } ;
```



