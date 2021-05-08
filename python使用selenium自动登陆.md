# python使用selenium自动登陆

## 安装
```bash
sudo pip3 install selenium
```
## 下载浏览器对应驱动
以Chrmoe浏览器为例
    Chrome: https://sites.google.com/a/chromium.org/chromedriver/downloads
将下载的文件解压，并放到系统路径下

```bash
sudo cp chromedriver /usr/bin/
```

## 使用
需要注意，uos浏览器是在chrome基础上封装了一层，找不到默认的chrome执行程序，需要单独指定路径
```bash
from selenium import webdriver

options = webdriver.ChromeOptions()
options.binary_location = "/usr/share/uosbrowser/uos-browser"
driver = webdriver.Chrome(options=options)
driver.get("https://www.baidu.com")
```

## 高级使用
总是要弹出浏览器，感觉不是很好，现在有一种新方法，不弹出浏览器即（headless）模式，默认在后台操作，完全自动化。后续功能自己实现了，此处只是展示自动登陆
使用google-chrome浏览器实现代码部分如下：

```bash
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
options = Options()

options.binary_location = "/opt/google/chrome/google-chrome"

options.add_argument('--headless') #核心部分
options.add_argument('--disable-gpu')
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')

browser = webdriver.Chrome(options=options)
browser.implicitly_wait(2)

browser.get("https://pms.uniontech.com")

username = browser.find_element_by_name("account")
username.send_keys('*****')

# 推荐使用by_xpath方式，关于xpath路径，可以在开发者页面中选中模块，然后右键复制
password = browser.find_element_by_xpath("//*[@id=\"loginPanel\"]/div/div[2]/form/table/tbody/tr[2]/td/input")
password.send_keys('*****')

login_button = browser.find_element_by_xpath('//*[@id="submit"]')
login_button.click()
```
