<!--markdown-->```
#!/usr/bin/bash

# typecho博客的数据库和保存文章的表
DATABASE='typecho'
CONTENTS="typecho_contents"

# mysql数据库的用户名和密码
USER=""
PASS=""
# mysql有写入权限的文件夹
MYSQLDIR='/var/lib/mysql'

# 博客备份到这里
BACKUPDIR='/home/mysql/blog'

# 这两个文件是记录数据库和文章更新的时间
UPDATEFILE="$BACKUPDIR/last_update_time.log"
MODFILE="$BACKUPDIR/modified.txt"


if [ ! -d $BACKUPDIR ]; then
\	mkdir $BACKUPDIR
fi

if [ ! -d $MYSQLDIR ]; then
\	mkdir $MYSQLDIR\	
fi

cd $BACKUPDIR

if [ ! -e $MODFILE ]; then
\	touch $MODFILE
fi

function check_update_time {

\	UPDATE_TIME=$(mysql -u$USER -p$PASS -e "select update_time from information_schema.tables where table_name='$CONTENTS';" | tail -n 1)

\	if [ -e "$UPDATEFILE" ]; then
\	\	LAST_UPDATE_TIME=$(cat $UPDATEFILE)
\	else
\	\	LAST_UPDATE_TIME='TIME_NULL'
\	fi

\	if [ "$UPDATE_TIME" = "$LAST_UPDATE_TIME" ]; then
\	\	return 0
\	else
\	\	echo "$UPDATE_TIME" > "$UPDATEFILE"
\	\	return 1
\	fi
}

function move_to_backupdir {
\	if [ -e $FILENAME ]; then
\	\	rm -f $FILENAME
\	fi
\	mysql -u$USER -pPASS -e "select text from $DATABASE.$CONTENTS where cid=$CID into outfile '$FILENAME' lines terminated by ''"

\	mv $FILENAME $BACKUPDIR/
}


check_update_time
if [ $? = "1" ]; then
\	CIDS=$(mysql -u$USER -pPASS -e "select cid from $DATABASE.$CONTENTS;" | tail -n +2)
\	for CID in $CIDS; do
\	\	TITLE=$(mysql -u$USER -p$PASS -e "select title from $DATABASE.$CONTENTS where cid=$CID" | tail -n 1 | sed 's/[ \\/]/-/g')
\	\	FILENAME="$MYSQLDIR/$CID-$TITLE.md"

        # 把所有的文章保存到列表
        POSTLIST[$CID]="$CID-$TITLE.md"

\	\	modtime=$(mysql -u$USER -p$PASS -e "select modified from $DATABASE.$CONTENTS where cid=$CID" | tail -n 1)
\	\	log_modtime=$(grep "^$CID " $MODFILE | awk '{print $2}')
\	\	if [ "$log_modtime" = "" ]; then
\	\	\	echo "$log_modtime,$modtime,  New file: $FILENAME"
\	\	\	move_to_backupdir
\	\	\	echo "$CID $modtime" >> $MODFILE
\	\	elif [ ! "$log_modtime" = "$modtime" ]; then
\	\	\	echo "$log_modtime,$modtime,  Modified: $FILENAME"
\	\	\	move_to_backupdir
\	\	\	sed -i "s/$CID .*/$CID $modtime/" $MODFILE
\	\	else
\	\	\	echo "$log_modtime,$modtime,  No change CID:$CID"
\	\	fi
\	done
    
    # 遍历文件，把不在POSTLIST中的文章删除
    for file in *.md; do
        delete="1"
        for post in ${POSTLIST[*]}; do
            if [ $file = $post ]; then
                delete="0"
            fi
        done
        if [ $delete = "1" ]; then
            rm -f $file
\	    echo "$file DELETE!"
        fi
    done 
\	
\	echo "git add --all"
\	git add --all

\	ehco "git commit"
\	git commit -m "update by script"

\	echo "git push"
\	git push

\	echo "Backup Done"
else
\	echo "database is not modified, exit"

fi
```	