---
layout:     post
title:      ReactiveCocoa 进阶
subtitle:   函数式编程框架 ReactiveCocoa 进阶
date:       2017-01-06
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - ReactiveCocoa
    - 函数式编程
    - 开源框架
---
`RequestViewModel.h`

```
#import <Foundation/Foundation.h>

@interface Book : NSObject

@property (nonatomic, copy) NSString *subtitle;
@property (nonatomic, copy) NSString *title;
@property (nonatomic, copy) NSString *image;

@end

@interface RequestViewModel : NSObject

// 请求命令
@property (nonatomic, strong, readonly) RACCommand *reuqesCommand;

//模型数组
@property (nonatomic, strong) NSArray *models;


@end
```

`RequestViewModel.m`

```
#import "RequestViewModel.h"

@implementation Book

- (instancetype)initWithValue:(NSDictionary *)value {
    
    if (self = [super init]) {
        
        self.title = value[@"title"];
        self.subtitle = value[@"subtitle"];
        self.image = value[@"image"];
    }
    return self;
}

+ (Book *)bookWithDict:(NSDictionary *)value {
    
    return [[self alloc] initWithValue:value];
}



@end

@implementation RequestViewModel

- (instancetype)init
{
    if (self = [super init]) {
        
        [self initialBind];
    }
    return self;
}


- (void)initialBind
{
    _reuqesCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        
      RACSignal *requestSiganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
          
          NSMutableDictionary *parameters = [NSMutableDictionary dictionary];
          parameters[@"q"] = @"悟空传";
          
          //
          [[AFHTTPSessionManager manager] GET:@"https://api.douban.com/v2/book/search" parameters:parameters progress:^(NSProgress * _Nonnull downloadProgress) {
              
              NSLog(@"downloadProgress: %@", downloadProgress);
          } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
              
              // 数据请求成功就讲数据发送出去
              NSLog(@"responseObject:%@", responseObject);
              
              [subscriber sendNext:responseObject];
              
              [subscriber sendCompleted];
              
          } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
              
              NSLog(@"error: %@", error);
          }];
          
          
         return nil;
      }];
        
        // 在返回数据信号时，把数据中的字典映射成模型信号，传递出去
        return [requestSiganl map:^id(NSDictionary *value) {
            
            NSMutableArray *dictArr = value[@"books"];
            
            NSArray *modelArr = [[dictArr.rac_sequence map:^id(id value) {
                
                return [Book bookWithDict:value];
                
            }] array];
            
            return modelArr;
            
        }];
        
    }];
}


@end

```

>最后附上GitHub：<https://github.com/qiubaiying/ReactiveCocoa_Demo>
