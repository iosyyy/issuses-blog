# [egg.js](https://github.com/iosyyy/issuses-blog/issues/10)

1. egg.js的逻辑

   egg.js主要的项目是在router中调用controller中的方法然后调用service中的方法最后从service调用底层数据库model.model负责定义表的结构.

2. 配置:

   ```jsx
   const config: PowerPartial<EggAppConfig> = {
       sequelize: {
         dialect: 'mysql',
         host: process.env.DEV_DB_HOST,// 数据库的地址
         port: 3306,
         database: 'graduate',// database name
         username: process.env.DEV_DB_USERNAME,// 数据库登录的用户名
         password: process.env.DEV_DB_KEY,// 数据库登录的密码
         define: {
           timestamps: true,// 时间戳默认会在插入的时候生成create_time
           underscored: true,
           freezeTableName: true,// 不然默认生成s结尾的tablename
         },
         query: {
           nest: true,
         },
       },
       multipart: {
         fileExtensions: ['.xlsx']// egg.js 默认不允许上传.xlsx文件此操作可以限制上传的文件
       }
     };
   ```

3. 表的结构如何定义:

   ```jsx
   module.exports = {
     up: async (queryInterface, Sequelize) => {
       const { INTEGER, STRING, DATE } = Sequelize;
       await queryInterface.createTable('merge_major_project', {
         id: { type: INTEGER, primaryKey: true, autoIncrement: true },
         major_id: {
           type: Sequelize.DataTypes.INTEGER,
           references: {
             model: {
               tableName: 'majors',
             },
             key: 'id',
           },
         },
         project_id: {
           type: Sequelize.DataTypes.INTEGER,
           references: {
             model: {
               tableName: 'projects',
             },
             key: 'id',
           },
         },
         created_at: DATE,
         updated_at: DATE,
       });
     },
   
     down: async (queryInterface, Sequelize) => {
       /**
        * Add reverting commands here.
        *
        * Example:
        * await queryInterface.dropTable('users');
        */
       await queryInterface.dropTable('merge_major_project');
     },
   };
   ```

   使用如上方法即可定义一个`merge_major_project`表值得注意的是`egg.js`会默认给表加上一个s作为结尾,在设置中修改`freezeTableName: true`即可完成表名自由

4. 多对多怎么设置

   ```jsx
   Projects.associate = () => {
     // foreignKey和otherKey都是中间表的属性分别对应,关联表和被关联表,targetKey还是被关联表的属性,sourceKey是关联表中属性
     app.model.Projects.belongsToMany(app.model.Majors, {
       through: app.model.MergeMajorProject,
       otherKey: 'major_uuid',
       sourceKey: 'uuid',
       foreignKey: 'project_uuid',
       targetKey: 'uuid',
     });
     // foreignKey代表关联表(.之前的乃个表)中的属性,target代表被关联的表中的属性
     app.model.Projects.belongsTo(app.model.Enterprises, {
       foreignKey: 'enterprise_uuid',
       targetKey: 'uuid',
     });
   };
   ```

5. 如何查询

   ```jsx
   transaction = await this.ctx.model.transaction();
   const results = await this.ctx.model.Projects.findAndCountAll({
     where,
     transaction,// 绑定egg.js中的事物
     distinct: true,// 去重保证count值的正确性
     include: [
       {
         model: this.ctx.model.Teachers,
         attributes: ['id', 'username', 'email', 'institute'],
         required: true,// true为inner join ,false为left join
         through: {
           where: {
             state: { [Op.like]: `%${state}%` }, // 通过through判断中间表的属性
           },
         },
         include: [
           {
             model: this.ctx.model.Schools,
           },
         ],
       },
       {
         model: this.ctx.model.Enterprises,
       },
       {
         model: this.ctx.model.Majors,
         attributes: ['name'],// 限制输出的行为name一条
       },
     ],
     offset: Number((page - 1) * 10),
     limit: 10,
     subQuery: false, // 不让在子查询里分页，全局处理,可以防止through:where中的一些问题
   });
   console.log(results.rows,results.count);
   ```