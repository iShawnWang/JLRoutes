JLRoutes
========

## JLRoutes 2.x 起不支持 iOS7 

从 1.x copy 代码, 来支持iOS7

```objective-c

if(![self isiOS7Later]){
            self.pathComponents = [self JLRoutes_parsePathComponentsiOS7];
            self.queryParams= [self JLRoutes_URLParameterDictionaryWithQueryStringiOS7:self.URL.query];
            return self;
}

... ...

#pragma mark - iOS7 Support
- (NSDictionary *)JLRoutes_URLParameterDictionaryWithQueryStringiOS7:(NSString*)queryStr
{
    NSMutableDictionary *parameters = [NSMutableDictionary dictionary];
    
    if (queryStr.length && [queryStr rangeOfString:@"="].location != NSNotFound) {
        NSArray *keyValuePairs = [queryStr componentsSeparatedByString:@"&"];
        for (NSString *keyValuePair in keyValuePairs) {
            NSArray *pair = [keyValuePair componentsSeparatedByString:@"="];
            // don't assume we actually got a real key=value pair. start by assuming we only got @[key] before checking count
            NSString *paramValue = pair.count == 2 ? pair[1] : @"";
            // CFURLCreateStringByReplacingPercentEscapesUsingEncoding may return NULL
            parameters[pair[0]] = [self JLRoutes_URLDecodedString:paramValue] ?: @"";
        }
    }
    return parameters;
}

- (NSString *)JLRoutes_URLDecodedString:(NSString*)str
{
    NSString *input = [str stringByReplacingOccurrencesOfString:@"+" withString:@" " options:NSLiteralSearch range:NSMakeRange(0, str.length)];
    return [input stringByRemovingPercentEncoding];
}

-(NSArray*)JLRoutes_parsePathComponentsiOS7
{
    NSPredicate *filterSlashesPredicate = [NSPredicate predicateWithFormat:@"NOT SELF like '/'"];
    NSArray *pathComponents = [(self.URL.pathComponents ?: @[]) filteredArrayUsingPredicate:filterSlashesPredicate];
    if ([self.URL.host rangeOfString:@"."].location == NSNotFound && ![self.URL.host isEqualToString:@"localhost"]) {
        // For backward compatibility, handle scheme://path/to/resource as if path was part of the path
        pathComponents = [@[self.URL.host] arrayByAddingObjectsFromArray:pathComponents];
    }
    return pathComponents;
}

-(BOOL)isiOS7Later
{
    return floor(NSFoundationVersionNumber) > NSFoundationVersionNumber_iOS_7_1;
}
```
