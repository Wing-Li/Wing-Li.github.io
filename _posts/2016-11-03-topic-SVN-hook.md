---
title: SVN上传时，同时将代码上传服务器——hook机制
excerpt: 多人WEB开发
categories:
  - topics
tag: PHP  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

这里是利用了 SVN 的一个功能——hook。

## 关于 SVN HOOK
#### HOOK 是干嘛的？
为了方便管理员控制提交的过程，Subversion 提供了 hook 机制。**当特定的事件发生时，相应的 hook 会被调用， hook 其实就相当于特定事件的处理函数。**每个 hook 会得到与它所处理的事件相关的参数，根据 hook 的返回值， Subversion 会决定是否继续当前的提交过程。

#### 当前 Subversion 提 供了 5 种可以安装的 hook ：

![](http://upload-images.jianshu.io/upload_images/1689895-6b068ae7b5cfc524.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***

## 需求：
多人开发 WEB 项目时，如果用的 SVN ，都会遇到这么一种情况：我不仅要把代码提交到 SVN，还需要把代码反应到服务器。
怎么办呢？难道每次我改完代码要立马看效果，都必须去上传一次服务器吗？

由此，我们就来解决这个问题。利用的就是 SVN 的 hook 钩子机制。

## 我用的是 Windows ,开发语言是 PHP，用的 laravel 框架。

先来说步骤：

> 1. 创建一个新的laravel项目，用来执行上传，添加自定义的上传命令；
2. 创建一个 hook.bat，来执行命令；
3. 设置 SVN 的 hook。

来，根据步骤，开始实现。

***

#### 1. 创建一个新的laravel项目，用来执行上传，添加自定义的上传命令
我的是 laravel 4.2 的框架，
5.0以上的看这里：https://laravel-china.org/docs/5.0/commands
都是类似的。

在4.2的项目如下：


**首先**，在 app\config\remote.php 文件下做一些配置，设置所要上传服务器的必要信息。

        'connections' => array(

            'dev' => array(
                'host'      => '172.0.0.1',
                'username'  => 'dev-user',
                'password'  => '',
                'key'       => 'E:\PHPProject\test\app\config\pk\generate_dev-key.pem',
                'keyphrase' => '123456',
                'root'      => '/home/dev-user',
            ),

            'stage'=>array(
                'host'      => '172.0.0.2',
                'username'  => 'stage-user',
                'password'  => '',
                'key'       => 'E:\PHPProject\test\app\config\pk\generate_dev-key.pem',
                'keyphrase' => '123456',
                'root'      => '/home/stage-user',
            ),
            'prod'=>array(
                'host'      => '172.0.0.3',
                'username'  => 'root',
                'password'  => '123456',
                'key'       => '',
                'keyphrase' => '',
                'root'      => '/var/www/html/project_prod',
            ),

        ),


**然后**，自定义上传的命令，在 app\commands 目录下创建 synchronize.php 文件。

        <?php

        use Illuminate\Console\Command;
        use Symfony\Component\Console\Input\InputOption;
        use Symfony\Component\Console\Input\InputArgument;

        class synchronize extends Command {

            /**
             * 第一个路径：是我们本地的路径，并不是全路径，会自动匹配。
             *
             * 第二个路径：是目标服务器的路径
             */
            private $app_path=[
                                'dev'=>["Trunk/Project_dev","/home/project_dev"],
                                'stage'=>["Trunk/Project_stage","/home/project_stage"],
                                'prod'=>["trunk/Project_prod","/var/www/html/project_prod"]
                              ];

            /**
             * The console command name.
             *
             * @var string
             */
            protected $name = 'synchronize';

            /**
             * The console command description.
             *
             * @var string
             */
            protected $description = 'synchronize dev source with svn';

            /**
             * Create a new command instance.
             *
             * @return void
             */
            public function __construct()
            {
                parent::__construct();
            }

            /**
             * Execute the console command.
             *
             * @return mixed
             */
            public function fire()
            {
                //
                set_time_limit(0);
                $path=$this->argument("path");
                $fp=fopen($path,"r");
                while(!feof($fp) && $line=fgets($fp))
                {
                    //去除换行符
                    $line=str_replace(PHP_EOL,'',$line);
                    //只对代码目录起效
                    array_walk($this->app_path,function($v,$k) use ($line){
                        $path_array=explode($v[0],$line,2);
                        if(count($path_array)==2)
                        {
                            \Log::info($line);
                            \Log::info($k);
                            //获取远程路径
                            $temp_remote_path=$v[1].$path_array[1];
                            if(is_dir($line))
                            {
                                //文件夹
                                \Log::info('cd '.$v[1]);
                                \Log::info('mkdir -p '.trim($path_array[1],'/'));
                                \SSH::into($k)->run(
                                    [
                                        'cd '.$v[1],
                                        'mkdir -p '.trim($path_array[1],'/'),
                                    ]
                                );
                            }else
                            {
                                //获取需要删除或者新建的目录或者文件
                                $re_path=explode('/',$path_array[1]);
                                $re_path_length=count($re_path);
                                $work_path=$v[1];
                                array_walk($re_path,function($v,$k) use ($re_path_length,&$work_path){
                                    if($k!=$re_path_length-1) $work_path.=$v.'/';
                                });
                                \Log::info(is_file($line));
                                if(is_file($line))
                                {
                                    //创建文件夹
                                    //文件夹
                                    \Log::info('mkdir -p '.$work_path);
                                    \SSH::into($k)->run(
                                        [
                                            'mkdir -p '.$work_path,
                                        ]
                                    );
                                    //上传文件
                                    \SSH::into($k)->put($line,$temp_remote_path);
                                }else
                                {
                                    //删除操作
                                    \Log::info('cd '.$work_path);
                                    \Log::info('rm -rf '.$re_path[$re_path_length-1]);
                                    \SSH::into($k)->run(
                                        [
                                            'cd '.$work_path,
                                            'rm -rf '.$re_path[$re_path_length-1],
                                        ]
                                    );
                                }
                            }
                        }
                    });
                }
                fclose($fp);
            }

            /**
             * Get the console command arguments.
             *
             * @return array
             */
            protected function getArguments()
            {
                return array(
                    array('path', InputArgument::REQUIRED, 'An example argument.'),
                );
            }

            /**
             * Get the console command options.
             *
             * @return array
             */
            protected function getOptions()
            {
                return array(
                    array('example', null, InputOption::VALUE_OPTIONAL, 'An example option.', null),
                );
            }

        }

**这个做完之后，这个项目就可以往我们指定的服务器传文件了，但是怎么才能让它执行呢？我们继续。**

<br />

#### 2. 创建一个 hook.bat，来执行命令
因为我用的是 Windows，所以，创建一个 .bat 的执行文件。

**首先**，在一个目录创建一个 .txt 的文本文件，输入如下：

        cd E:\PHPProject\test
        D:\wamp\bin\php\php5.5.12\php.exe artisan synchronize %1

第一行的路径是我们上一步创建的项目的所在路径；
第二行是php的安装路径。

**然后**，再把这个 .txt 文件重命名，将其后缀名 改为 .bat。

**这样，我们就建好了一个钩子(hook)，最后就是将其配置到 SVN。**

<br />

#### 3. 设置 SVN 的 hook
如图，很明了。


![点击 Setting](http://upload-images.jianshu.io/upload_images/1689895-a7369943402171e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![添加](http://upload-images.jianshu.io/upload_images/1689895-77b9fdd1d0cd163c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![](http://upload-images.jianshu.io/upload_images/1689895-a7a5d7448bb57836.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br />
大功告成。
***

<br />
这样，之后我们每次提交 SVN 的时候，就会自动同步到我们所配置的服务器。
这里需要提一下的是，在我们第一步所创建的项目，里面也会记录上传的 LOG，在 app\storage\logs\laravel.log 文件记录。

<br />
好了，文章到此结束。

如果有什么问题，请一定指出。