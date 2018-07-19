``` js
var initProxy
var allowedGlobals = makeMap('Infinity,undefined,NaN,isFinite,isNaN,' +
  'parseFloat,parseInt,decodeURI,decodeURIComponent,encodeURI,encodeURIComponent,' +
  'Math,Number,Date,Array,Object,Boolean,String,RegExp,Map,Set,JSON,Intl,' +
  'require' // for Webpack/Browserify
)

var warnNonPresent = function (target, key) {
  warn(
      "Property or method \"" + key + "\" is not defined on the instance but " +
      'referenced during render. Make sure that this property is reactive, ' +
      'either in the data option, or for class-based components, by ' +
      'initializing the property. ' +
      'See: https://vuejs.org/v2/guide/reactivity.html#Declaring-Reactive-Properties.',
      target
  )
}

var hasProxy = typeof Proxy !== 'undefined' && Proxy.toString().match(/native code/)

if (hasProxy) {
  var isBuiltInModifier = makeMap('stop,prevent,self,ctrl,shift,alt,meta,exact')
  config.keyCodes = new Proxy(config.keyCodes, {
    set: function set (target, key, value) {
      if (isBuiltInModifier(key)) {
        warn(("Avoid overwriting built-in modifier in config.keyCodes: ." + key));
        return false
      } else {
        target[key] = value
        return true
      }
    }
  })
}
var hasHandler = {
  has: function has (target, key) {
    var has = key in target
    var isAllowed = allowedGlobals(key) || key.charAt(0) === '_'
    if (!has && !isAllowed) {
      warnNonPresent(target, key)
    }
    return has || !isAllowed
  }
}

var getHandler = {
  get: function get (target, key) {
    if (typeof key === 'string' && !(key in target)) {
      warnNonPresent(target, key)
    }
    return target[key]
  }
}

initProxy = function initProxy (vm) {
  if (hasProxy) {
    // determine which proxy handler to use
    var options = vm.$options
    var handlers = options.render && options.render._withStripped
      ? getHandler
      : hasHandler
    vm._renderProxy = new Proxy(vm, handlers)
  } else {
    vm._renderProxy = vm
  }
}