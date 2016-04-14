# Mysql存储过程使用多个游标的处理

## 定义数据库表

```sql
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `score` varchar(255) DEFAULT NULL,
  `update_time` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  `create_time` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES ('1', 'ronnie', '100', '2016-04-14 10:44:01', '2016-04-14 10:44:06');
INSERT INTO `student` VALUES ('2', 'john', '95', '2016-04-14 10:44:22', '2016-04-14 10:44:25');
INSERT INTO `student` VALUES ('3', 'mark', '90', '2016-04-14 10:44:32', '2016-04-14 10:44:35');
```

## 定义测试用存储过程

```sql
CREATE PRODCEDURE loop_test(OUT count INT)
BEGIN

	DECLARE name VARCHAR(20);
	DECLARE score VARCHAR(20);
	DECLARE number INT;
	DECLARE done INT DEFAULT FALSE;
	DECLARE cur CURSOR for SELECT name, score from student;
	DECLARE cur2 CURSOR for SELECT name, score from student;
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

	SET number = 0;

	OPEN cur;
	loop1: LOOP
		FETCH cur into name, score;

		IF done THEN
			LEAVE loop1;
		END IF;

		SET number = number + 1;
	END LOOP;

	CLOSE cur;

	SET done = FALSE;//两个游标的情况下，注意在遍历第二个游标之前将done标志设为FALSE

	OPEN cur2;
	loop2: LOOP
		FETCH cur2 into name, score;

		IF done THEN
			LEAVE loop2;
		END IF;

		SET number = number + 1;
	END LOOP;

	CLOSE cur2;

	SET count = number;
END
```

## 调用存储过程

```sql
CALL test_loop(@count);
SELECT @count;
```

## 查看结果

结果为6
