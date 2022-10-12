```java
public class CodeGet {

    public static void main(String[] args) {
        String pathPrefix = System.getProperty("user.dir");
        String model = "";
        String parentPackageName = "cn.walls1717";
        String modelName = "mybatisplus";
        String dbUrl = "localhost:3306";
        String db = "mybatisplus";
        String dbUsername = "root";
        String dbPassword = "chengjie";
        String tablePrefix = "";
        String[] tables = {"student"};
        String author = "walls1717";
        doGet(pathPrefix, model, parentPackageName, modelName, dbUrl, dbUsername, dbPassword, db, tablePrefix, tables, author);
    }

    /**
     * 代码生成器
     * 
     * @param pathPrefix 输出路径前缀，这个是获取当前路径。 例如（ CodeGet.class在目录
     *            E:\WorkStation\program\studyProgram\mybatisPlus\mybatisPlus_demo\src\test\java
     *            System.getProperty("user.dir")获取的路径则为 E:\WorkStation\program\studyProgram\mybatisPlus\mybatisPlus_demo
     *            ）
     * @param model 模块名，没有拆分模块可以为空
     * @param parentPackageName 父包名，后面跟modelName生成全包名
     * @param modelName 模块名，用于生成包名和mapper文件路径
     * @param dbUrl 数据库ip与端口（localhost:3306）
     * @param dbUsername 数据库用户名
     * @param dbPassword 数据库用户密码
     * @param db 数据库名
     * @param tablePrefix 数据库表前缀，用于排除表前缀，没有可以为空
     * @param tables 数据库表名，对这些表生成类
     * @param author 作者名，生成类的doc注解（@author author）
     */
    public static void doGet(String pathPrefix, String model, String parentPackageName, String modelName, String dbUrl,
        String dbUsername, String dbPassword, String db, String tablePrefix, String[] tables, String author) {

        FastAutoGenerator
            .create("jdbc:mysql://" + dbUrl + "/" + db + "?serverTimezone=Asia/Shanghai", dbUsername, dbPassword)
            // 全局配置
            .globalConfig(builder -> {
                // 设置作者
                builder.author(author)
                    // 覆盖已生成的文件
                    .fileOverride()
                    // 设置输出路径
                    .outputDir(pathPrefix + model + "/src/main/java")
                    // 生成后不打开
                    .disableOpenDir()
                    // 设置javadoc注释日期格式 @since yyyy/MM/dd hh:mm
                    .commentDate("yyyy/MM/dd HH:mm");
            })
            // 包配置
            .packageConfig(builder -> {
                // 设置父包名
                builder.parent(parentPackageName)
                    // 设置父包模块名
                    .moduleName(modelName)
                    // 设置mapperXml生成路径
                    .pathInfo(Collections.singletonMap(OutputFile.mapperXml,
                        pathPrefix + model + "\\src\\main\\resources\\mapper\\" + modelName));
            }).strategyConfig(builder -> {
                // 排除表前缀
                builder.addTablePrefix(tablePrefix);
                // 要生成的表
                builder.addInclude(tables)
                    // 服务生成策略
                    .serviceBuilder()
                    // 服务名
                    .formatServiceFileName("%sService")
                    // 服务实现类名
                    .formatServiceImplFileName("%sServiceImpl")

                    // 实体生成策略
                    .entityBuilder()
                    // 添加lombok注解
                    .enableLombok()
                    // 表字段下划线转驼峰
                    .columnNaming(NamingStrategy.underline_to_camel)

                    // 控制器生成策略
                    .controllerBuilder()
                    // 开启RestFul风格
                    .enableRestStyle().enableHyphenStyle()

                    // mapper生成策略
                    .mapperBuilder()
                    // 生成基础resultMap
                    .enableBaseResultMap();
            })
            // 使用Freemarker引擎模板，默认的是Velocity引擎模板
            // 使用Velocity模板的话entity类格式上有bug
            .templateEngine(new FreemarkerTemplateEngine()).execute();
    }
}
```

