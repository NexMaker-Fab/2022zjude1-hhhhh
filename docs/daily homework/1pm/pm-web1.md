
# 0.First and foremost
There are many ways to build a website. We use "docsify" here, which is a magic tool for generating document websites.   
The main tutorial we refer to is this: [the main tutorial](https://www.nexmaker.com/doc/1projectmanage/github&docsify.html)
# 1.Some preparatory work
## 1.1 Install `node.js`
Before installing `docsify`, we need to install the `npm` package manager, and the installation of `node.js` will automatically install `npm`.
+ Installation  
>Download the installation program from [the official website](https://nodejs.org/en/).  
>Double click the downloaded exe to install it.Next step is the next step until it is completed.
+ verification
>Open the `cmd`command line and enter `node -v`.  
>If the node version is displayed, the nodejs installation is successful, as shown in the following figure.  
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110105804.png)  

## 1.2 Install `docsify`
>Enter the following in cmd.exe:
>``` npm i docsify-cli -g```

## 1.3 Make sure the position and then initialize environment
+ Enter the file location from `cmd.exe`
> You need to create a project folder first, here I use the folder cloned from `github repository`[Tutorial step2](https://www.nexmaker.com/doc/1projectmanage/github&docsify.html).  
> Then Enter the `cd` command into the folder path in `cmd.exe`. You can also drag the folder into the exe, and it will automatically generate the path, as shown in the following figure. 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110120339.png)
+ Initialize
>Enter the following in cmd.exe:
>```docsify init ./docs```  
>After successful initialization, you can see several files created in the directory：  
>`index.html`:Entry File.  
>`README.md`:It will be rendered as the homepage content.  
>`.nojekyll`:Used to prevent GitHub Pages from ignoring files that begin with an underscore.

## 1.4 Preview
>Enter the following in cmd.exe:
>``` docsify serve docs```  
>You will get the following results.   
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110154062.png)  
>Then opern browser to visit http://localhost:3000, you will get a initial website. 

## 1.5 Image upload 
We use picgo+github to load images.
>The tutorial is at this [link](https://www.nexmaker.com/doc/1projectmanage/imageuploadservice.html).  
>What i want to emphasize is 2 points.  
>1.In the step to generate a new token ,the following boxes must be chosen, otherwise the images cannot be uploaded.  
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210111844398.png)  
>2.When the token has been generated, you must copy the token at time, otherwise the token would not be found again.

# 2.Setting `index.html`
Copy the following code into `index.html` for overwriting
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Peopl3 Drag Per2on</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="Description">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
  <!-- 侧边栏目录折叠 -->
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify-sidebar-collapse/dist/sidebar.min.css" />
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify-sidebar-collapse/dist/sidebar-folder.min.css" />
</head>
<body>
  <div id="app">Loading...</div>
  <script>
    window.$docsify = {
      name: 'Peopl3 Drag Per2on',
      repo: '',
      loadSidebar: true,  //prepare for sidebar
      loadNavbar: true,   //prepare for navbar
      coverpage: true,
      onlyCover: true,
      search: {
        noData: {
          '/': 'None!'
        },
        paths: 'auto',
        placeholder: {
          '/': 'Search...'
        }
      }
    }
  </script>
  <!-- Docsify v4 -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-sidebar-collapse/dist/docsify-sidebar-collapse.min.js"></script>
</body>
</html>
```
The sidebar and its contraction, cover page and search bar will be described in the following steps.

# 3.Setting sidebar
+ Configure sidebar
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110233085.png)  
>The code in the image above modify the `index.html` configuration file to configure the `loadSidebar` option.
+ create `_sidebar.md` 
>Create folders and files as shown in the picture:  
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110240469.png)  
>Copy the following code into `_sidebar.md`  
>See this link for usage of the `markdown` language: [link](https://www.runoob.com/markdown/md-image.html)  

```
<!-- 侧边栏 docs/_sidebar.md -->
- Team introduce
- [Yang Haohao](team%20introduce/Yang%20Haohao.md)
- [Tang Yinlin](team%20introduce/Tang%20Yilin.md)
- [Gu Yuxin](team%20introduce/Tang%20Yilin.md)
- [Leng Shiyang](team%20introduce/Leng%20Shiyang.md)
- [Wang Chen](team%20introduce/Wang%20Chen.md)
- Daily homework
- [1. PM](daily%20homework/1pm/pm-guidebook.md)
    - [How to build web](daily%20homework/1pm/pm-web1.md)
- [2. CAD]()
- [3. 3D printing]()
- Final project
- [Description](final%20project/description.md)
- [How to design](final%20project/how%20to%20design.md) 
- [How to make](final%20project/how%20to%20make.md)
```  
+ Sidebar contraction
>The codes in the following images configure the sidebar contraction and some interactive effects.  
>This comes from the open source code on github.[links](https://github.com/iPeng6/docsify-sidebar-collapse) 
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110302065.png)  
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110306220.png)

# 4.Setting coverpage
>Configure the coverpage parameter in `index.html` to open the coverpage. Usually, the cover page and the first page appear at the same time. After `onlyCover=true` is set, the cover page becomes an independent cover page.  
>The code settings in the following figure play such a role:  
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110312548.png)  

>Add the profile `_coverpage.md` to configure the coverpage, the code and rendering are as follows:  
```
# Peopl3 Drag Per2on  
## Welcome to the website of our team 
* TECHNOLOGY
* DESIGN
* AESTHETIC
[Get Started](./README.md)
```
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110317829.png)

# 5.Setting search bar
>Configure search bar in `index.html` as following  
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202210110319460.png)

# 6.Save all document and repeat step 1.4
# 7.Last but not least
Congratulation!  
You did it!