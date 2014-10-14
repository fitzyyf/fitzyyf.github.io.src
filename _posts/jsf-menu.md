title: "jsf-menu"
date: 2008-04-17 22:28
categories: 
- 编程语言
---

前段时间刚换公司，进入公司正好碰到一个项目在使用JSF框架，当时项目经理就让我做下基本的一些信息处理，也就是增删改查（刚去公司，加上能力又不照，只好从基本做起，呵呵）

但是在做的过程中，因为用户表与角色表是想关联的，而在显示所有用户的时候，需要将想对应的关联角色显示出来，而数据库中只是主键关联，并非名称关联。原本以为只需要在JSF页面上这样就可以了

{% codeblock button lang:html %}
<h:column>
        <h:dataTable border="1" id = "users" value="#{UserBean.list}" var="u" styleClass="orders" headerClass="ordersHeader" rowClasses="evenColumn,oddColumn"> 
        <f:facet name="header">
            <f:verbatim escape="true">角色</f:verbatim>
        </f:facet>
        <h:selectOneMenu value="#{u[1]}">
            <f:selectItems value="#{u[1]}"/>
        </h:selectOneMenu>
    </h:dataTable> 
</h:column>
{% endcodeblock %}

可是运行后，发现根本不可以显示角色。在网上找了下，终于找到解决办法了。

1. JSF页面

{% codeblock jsf-page lang:html %}
<h:column>
<h:dataTable border="1" id = "users" value="#{UserBean.list}" var="u" styleClass="orders" headerClass="ordersHeader" rowClasses="evenColumn,oddColumn"> 
                        <f:facet name="header">
                            <f:verbatim escape="true">角色</f:verbatim>
                        </f:facet>
                        <h:selectOneMenu value="#{u[1]}">
                            <f:selectItems value="#{#{RoleBean.roleList}}"/>
                        </h:selectOneMenu>
                    </h:column>
</h:dataTable> 
{% endcodeblock %}

2. 后台Bean

{% codeblock RoleBean.java lang:java %}
public class RoleBean {

    public int roleId;
    public String roleName;
    public String roleRemark;
    public String msg;
    public List<SelectItem> roleList = new LinkedList<SelectItem>();
    public Vector list;

    public RoleBean() throws Exception {
        if (roleList == null) {
            roleList = new LinkedList<SelectItem>();
        }
        RoleBR rbr = new RoleBR();
        Vector v = new Vector();
        String id = null;
        String name = null;
        v = rbr.getAllRoleName();
        list = rbr.getAllList();
        Iterator it = v.iterator();
        while (it.hasNext()) {
            String[] row = (String[]) it.next();
            for (int i = 0; i < row.length; i++) {
                if (i == (i / 2)) {
                    id = row[i].trim();
                    name = row[i + 1].trim();
                } else {
                    continue;
                }
                roleList.add(new SelectItem(id, name));
            }
        }
    }

    public int getRoleId() {
        return roleId;
    }

    public void setRoleId(int roleId) {
        this.roleId = roleId;
    }

    public String getRoleName() {
        return roleName;
    }

    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }

    public String getRoleRemark() {
        return roleRemark;
    }

    public void setRoleRemark(String roleRemark) {
        this.roleRemark = roleRemark;
    }

    /**
     * create the selectOneMenu for user regrest.jsp
     * @return
     */
    public List<SelectItem> getRoleList() {
        return roleList;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
{% endcodeblock %}

3. 数据库层DAO

{% codeblock roledao.java lang:java %}
public class RoleDA {
private DBBase theDB = new DBBase();//数据库连接
private String str_sql = null;
/**
     * 取得所有的角色
     * @return
     */
    public Vector getAllList() {
        Vector result = new Vector();
        str_sql = "select t.roleid,t.rolename,t.remark from s_role t";
        result = theDB.selectAll(str_sql);
        return result;
    }
}
{% endcodeblock %}

4. JSF配置文件

{% codeblock faces-xml lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>

<faces-config xmlns="http://java.sun.com/xml/ns/javaee"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-facesconfig_1_2.xsd"
              version="1.2">
<managed-bean>
        <managed-bean-name>RoleBean</managed-bean-name>
        <managed-bean-class>ahbagz.bean.RoleBean</managed-bean-class>
        <managed-bean-scope>session</managed-bean-scope>
    </managed-bean>
</faces-config>
{% endcodeblock %}
