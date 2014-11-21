// Inspired in github.com/paulirish/jquery-ajax-localstorage-cache and working 100% with $.Deferred and promise.
var supportsLocalStorage = function () { var a = "ajax-ls-cache"; try { return localStorage.setItem(a, a), localStorage.removeItem(a), !0 } catch (r) { return !1 } } ();

$.ajaxPrefilter(function (options, originalOptions, jqXHR) {
    // Cache it ?
    if (!supportsLocalStorage || !options.localCache) return;

    // Deprecation Notice: The jqXHR.success(), jqXHR.error(), and jqXHR.complete() callbacks are deprecated as of jQuery 1.8.
    // Still support success for now
    //jqXHR.done = jqXHR.done || jqXHR.success;

    var hourstl = options.cacheTTL || 8;
    var cacheKey = options.cacheKey || options.url.replace(/jQuery.*/, '') + options.type + (options.data || '');

    // isCacheValid is a function to validate cache
    if (options.isCacheValid && !options.isCacheValid()) { localStorage.removeItem(cacheKey) }
    // if there's a TTL that's expired, flush this item
    var ttl = localStorage.getItem(cacheKey + 'cachettl');
    if (ttl && ttl < +new Date()) {
        localStorage.removeItem(cacheKey);
        localStorage.removeItem(cacheKey + 'cachettl');
        ttl = 'expired';
    }

    var value = localStorage.getItem(cacheKey);

    if (value) {
        //In the cache? So get it, apply done callback & abort the XHR request
        // parse back to JSON if we can.
        if (options.dataType.indexOf('json') === 0) value = JSON.parse(value);

        jqXHR = $.Deferred(function (defer) {
            defer.resolveWith(this, [value]);            
            // Abort is broken on JQ 1.5 :(
            if (jqXHR && jqXHR.state() === 'pending') {
                //Calls any error / fail callbacks of jqXHR
                jqXHR.abort()
            }
        }).promise(jqXHR)
        jqXHR.success = jqXHR.done;
        jqXHR.error = jqXHR.fail;

    } else {

        //If it not in the cache, we change the done callback, just put data on localstorage and after that apply the initial callback
        jqXHR.done(function (data) {
            var strdata = data;
            if (options.dataType.indexOf('json') === 0) strdata = JSON.stringify(data);

            // Save the data to localStorage catching exceptions (possibly QUOTA_EXCEEDED_ERR)
            try {
                // store data
                localStorage.setItem(cacheKey, strdata);
                // store timestamp
                if (!ttl || ttl === 'expired') {
                    localStorage.setItem(cacheKey + 'cachettl', +new Date() + 1000 * 60 * 60 * hourstl);
                }
            } catch (e) {
                // Remove any incomplete data that may have been saved before the exception was caught
                localStorage.removeItem(cacheKey);
                localStorage.removeItem(cacheKey + 'cachettl');
                if (options.cacheError) options.cacheError(e, cacheKey, strdata);
            }
        });
    }
});
