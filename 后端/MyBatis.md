### `MyBatis` insert 后返回 id

在`mapper.xml`中的`insert`语句加入`useGeneratedKeys="true" keyProperty="id"`这两个属性，就会在插入后，把id设置会插入的bean对象中，比如这种：

```xml
<insert id="insertOne" parameterType="com.ruoyi.yw.domain.YwSafetyEducation" useGeneratedKeys="true" keyProperty="id">
    insert into yw_safety_education (<include refid="Base_Column_List"/>)
    values (#id, #educationTheme, #educationTimeStart, #educationTimeEnd, #signInFileUrl, #participantPhotoUrls)
</insert>
```