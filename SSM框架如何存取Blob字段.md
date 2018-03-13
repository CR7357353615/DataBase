# SSM框架如何存取Blob字段
---
Bean层：需要将待存储的内容类型定义为byte[]
```java
public class SysFiles {
  /**
  * 文件ID
  */
  private String fileId;
  /**
  * 文件类型 0-头像 1-电子发票影像 2-申报表
  */
  private Integer type;
  /**
  * 文件内容
  */
  private byte[] content;
}
```

Controller层：

存储时，将待存储文件转化为Inputstream流，再转为byte数组。set进对象中。
```java
SysFiles sysFiles = new SysFiles();
sysFiles.setFileId(defaultFileName);
//将Inputstream (is)转化为bytes并且压缩
byte[] bytes = FileCopyUtils.copyToByteArray(is);
sysFiles.setContent(bytes);
sysFiles.setType(0);
sysFileSerivce.insertFiles(sysFiles, null);
```

获取时，可以直接得到对象，并获得文件的byte[]类型形式。

Service层：
```java
@Service("sysFileService")
public class SysFileServiceImpl implements ISysFileService{
  @Autowired
  SysFilesMapper sysFilesMapper;

  @Override
  public int insertFiles(SysFiles sysFiles,String path) {
    return sysFilesMapper.insertSelective(sysFiles);
  }

  @Override
	public SysFiles selectFiles(String fileId) {
		return sysFilesMapper.selectByPrimaryKey(fileId);
	}
}
```

Mapper层：
```java
public interface SysFilesMapper {
  int insertSelective(SysFiles record);

  SysFiles selectByPrimaryKey(String fileId);
}
```

XML层：

Blob字段对应的jdbcType="LONGVARBINARY"
```xml
<resultMap id="BaseResultMap" type="cn.com.servyou.vtax.common.vo.SysFiles" >
  <id column="file_id" property="fileId" jdbcType="VARCHAR" />
  <result column="type" property="type" jdbcType="INTEGER" />
</resultMap>
<resultMap id="ResultMapWithBLOBs" type="cn.com.servyou.vtax.common.vo.SysFiles" extends="BaseResultMap" >
  <result column="content" property="content" jdbcType="LONGVARBINARY" />
</resultMap>

<insert id="insertSelective" parameterType="cn.com.servyou.vtax.common.vo.SysFiles" >
  insert into sys_files
  <trim prefix="(" suffix=")" suffixOverrides="," >
    <if test="fileId != null" >
      file_id,
    </if>
    <if test="type != null" >
      type,
    </if>
    <if test="content != null" >
      content,
    </if>
  </trim>
  <trim prefix="values (" suffix=")" suffixOverrides="," >
    <if test="fileId != null" >
      #{fileId,jdbcType=VARCHAR},
    </if>
    <if test="type != null" >
      #{type,jdbcType=INTEGER},
    </if>
    <if test="content != null" >
      #{content,jdbcType=LONGVARBINARY},
    </if>
  </trim>
</insert>

<select id="selectByPrimaryKey" resultMap="ResultMapWithBLOBs" parameterType="java.lang.String" >
  select
  <include refid="Base_Column_List" />
  ,
  <include refid="Blob_Column_List" />
  from sys_files
  where file_id = #{fileId,jdbcType=VARCHAR}
</select>
```
