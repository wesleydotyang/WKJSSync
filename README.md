# WKJSSync
JS call native by url is async in WK. This is a solve for sync.


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
