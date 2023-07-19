编译阶段修改xcode工程
https://blog.csdn.net/weixin_33785108/article/details/91430729
https://blog.csdn.net/sun1018974080/article/details/107641744
之后就我们要做的最后一步, 把之前的操作反过来再做一遍, 然后调用 save 方法保存 target
这里值得注意的是在处理多文件夹层级递归的时候, 我们需要生成新的对应的 group ,这里用到了 group 对象的几种不同的 path, 需要选择合适的属性加以利用.

我打开了我们工程的配置文件, 就是一个超级大的 hash 字典, 根节点有五个 archiveVersion, classes, objectVersion, objects, rootobject
其中 objects 是最重要的一个, 里面的每个子节点对应的就是一个个不同的配置文件, 他们每一个配置所对应的 key, 都是一个唯一的 UUID, 而 rootobjects 指向的就是这么一个 UUID
其中配置之前都有一段注释来解释配置的作用, 在配置内部也同样拥有一个 isa 字段来表示这个配置的类型, 那么我就直接无耻的搬运一下前辈们已经总结好的类型, 不完全统计,大概有如下一些类型

PBXBuildFile
PBXBuildPhase
PBXAppleScriptBuildPhase
PBXCopyFilesBuildPhase
PBXFrameworksBuildPhase
PBXHeadersBuildPhase
PBXResourcesBuildPhase
PBXShellScriptBuildPhase
PBXSourcesBuildPhase
PBXContainerItemProxy
PBXFileElement
PBXFileReference
PBXGroup
PBXVariantGroup
PBXTarget
PBXAggregateTarget
PBXLegacyTarget
PBXNativeTarget
PBXProject
PBXTargetDependency
XCBuildConfiguration
XCConfigurationList
总结
project = Xcodeproj::Project.open(path)
target = project.targets.first
group = project.main_group.find_subpath(File.join('DKNightVersion', 'Pod', 'Classes', 'UIKit'), true)
group.set_source_tree('SOURCE_ROOT')
file_ref = group.new_reference(file_path)
target.add_file_references([file_ref])
project.save
其实到现在为止, 我感觉到使用代码向 Xcodeproj 中添加文件是很简单的事情, 那是因为, 首先有 Xcodeproj 这样文档糟糕但是功能还是比较齐全的第三方框架, 而且这是我在几天不停地阅读源代码, 不停被坑, 一点一点尝试才摸索出来的结果, 都是泪啊…不想多说了…不过最后把这个问题解决之后, 自我感觉还是蛮好的还是挺高兴的. 嗯, 就这样.


https://github.com/luisrecuenco/FastXcodeFile/blob/master/fast_xcode_file.py
#!/usr/bin/python

import sys, os, time
from mod_pbxproj import XcodeProject

project = "__PROJECT_NAME__" # Only used in the file comment
files_group = "__FILES_GROUP__" # The Xcode group where you want the new files to be created
files_path = "__XCODE_FILES_PATH__" # The file system path where the new files will be created
xcode_proj_path = "__XCODE_PROJECT_PATH__"
super_class = "__SUPERCLASS__" # NSObject, UIViewController...
framework = "__FRAMEWORK__" ## Foundation, UIKit...
author = "__AUTHOR__"
class_name = sys.argv[1] # The class name will be the first argument of the script
header_file = "{0}.h".format(class_name)
implementation_file = "{0}.m".format(class_name)
header_full_file = "{0}/{1}".format(files_path, header_file)
implementation_full_file = "{0}/{1}".format(files_path, implementation_file)

def check_file(file):
    if os.path.exists(file):
        sys.exit()

def check_files_existence():
    check_file(header_full_file)
    check_file(implementation_full_file)

def create_files():
    try:
        date = time.strftime("%d/%m/%Y")
        copyright = "//\n//  {0}\n//  {1}\n//\n//  Created by {2} on {3}.\n//  Copyright (c) 2015, {1}. All rights reserved.\n//"
        header_copyright = copyright.format(header_file, project, author, date)
        implementation_copyright = copyright.format(implementation_file, project, author, date)

        # header file
        file = open(header_full_file,'w')
        file.write("{0}\n\n#import <{1}/{1}.h>\n\n@interface {2} : {3}\n\n@end\n".format(header_copyright, framework, class_name, super_class))
        file.close()

        # implementation file
        file2 = open(implementation_full_file,'w')
        file2.write("{0}\n\n#import \"{1}\"\n\n@implementation {2}\n\n@end\n".format(implementation_copyright, header_file, class_name))
        file2.close()

    except:
        print('Something went wrong')

def add_files_to_project():
    project = XcodeProject.Load('{0}/project.pbxproj'.format(xcode_proj_path))
    group = project.get_or_create_group(files_group)
    project.add_file_if_doesnt_exist(header_full_file, group)
    project.add_file_if_doesnt_exist(implementation_full_file, group)
    project.save()

check_files_existence()
create_files()
add_files_to_project()