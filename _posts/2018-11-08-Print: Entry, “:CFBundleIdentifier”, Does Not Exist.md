# 解决Print: Entry, “:CFBundleIdentifier”, Does Not Exist

用react-native开发安卓，回来运行ios报错

```js
Print: Entry, “:CFBundleIdentifier”, Does Not Exist
```

问题产生原因：

```js
/Users/你的用户名/.rncache中boost_1_63_0.tar.gz，double-conversion-1.1.5.tar.gz，folly-2016.09.26.00.tar.gz，glog-0.3.4.tar.gz文件不完整。或者node_modules/react-native/third-party 文件不完整。
```

解决办法：

```js
1、删除/user/你的用户名/.rncache目录下的boost_1_63_0。重新下载,下载网址http://www.boost.org/users/history/version_1_63_0.html

2、打开命令行工具，在项目目录下输入

rm -rf node_modules && rm -rf ~/.rncache && yarn

3、npm install 

4、react-native upgrade (这个我要重点说一下，只更新IOS的文件就行，全更新了，android的别给人家动。)

5、react-native run-ios
```



