# WKJSSync
JS call native by url is async in WKWebView. This is a solve for sync.


in `WKViewController`

`
self.webView.UIDelegate = self;
`

<pre>
-(void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable))completionHandler
{
    //listen on js prompt func. About 0.4ms interaction time on simulator
    if ([prompt isEqualToString:@"SOME_KEY_WORD"]) {
        NSString *result = [YOURJSHANDLER handleJSPromptCall:defaultText];
        //return result
        completionHandler(result);
    }else{//default prompt
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:prompt message:nil preferredStyle:UIAlertControllerStyleAlert];
        [alert addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
            textField.text = defaultText;
        }];
        [alert addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
            completionHandler(nil);
        }]];
        [alert addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
            NSString *text = [alert.textFields.firstObject text];
            completionHandler(text);
        }]];
        [self presentViewController:alert animated:YES completion:nil];
    }
    
}
</pre>

Inject JS below to WKWebView on MainFrameStart
<pre>
var app = {};
app.invokeNative = function(methodAndArg){
var result=prompt('YOUR_PROMPT_KEYWORD',JSON.stringify(methodAndArg)); //This will call native runJavaScriptTextInputPanelWithPrompt
//asume APP returned JSONString, which contains errorMsg or payload return value
var resultObj=JSON.parse(result);
if(resultObj.errorMsg) {//error handling
    var error = new Error(resultObj.errorMsg);
    error.code = resultObj.errorCode;
    throw error;
} else {
    return resultObj.payload;
}

//define your global functions
window.demoFunc = function(){
    return app.invokeNative({'"funcName":window.demoFunc',"args":arguments})
}
...

</pre>

Handle JS 'invokeNative' function and return value for functions you defined
<pre>
...
</pre>

Finally,in JS simply write code below to call native and get some return value
<pre>
var res = window.demoFunc('a')
<pre>

