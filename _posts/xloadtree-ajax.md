title: "XloadTree实现Ajax树"
date: 2008-08-04 09:56
tags: 
- xloadtree
categories:  
- 开发笔记
---
前不久，项目中需要使用到树，本来是使用的是JSFmyfaces的tree2控件（项目是JSF框架的），但是最近，可能由于代码问题，树经常报错（数组过长），刷新就可以了，故想换个树，在G上搜索一翻，发现XLOADTREE比较不错。就先写了个Demo，以下是代码：

{% codeblock jsp lang:html%}
<%-- 
    Document   : index
    Created on : 2008-7-31, 15:31:04
    Author     : 木易有峰
--%>

<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">

<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <script   type="text/javascript"   src="resources/xtree.js"></script>
        <script   type="text/javascript"   src="resources/xmlextras.js"></script>
        <script   type="text/javascript"   src="resources/xloadtree.js"></script>
        <link   type="text/css"   rel="stylesheet"   href="resources/xtree.css"   />
        <title>JSP Page</title>
    </head>
    <body>

        <script type="text/javascript">
            /// XP Look
            webFXTreeConfig.rootIcon        = "images/xp/folder.png";
            webFXTreeConfig.openRootIcon    = "images/xp/openfolder.png";
            webFXTreeConfig.folderIcon        = "images/xp/folder.png";
            webFXTreeConfig.openFolderIcon    = "images/xp/openfolder.png";
            webFXTreeConfig.fileIcon        = "images/xp/file.png";
            webFXTreeConfig.lMinusIcon        = "images/xp/Lminus.png";
            webFXTreeConfig.lPlusIcon        = "images/xp/Lplus.png";
            webFXTreeConfig.tMinusIcon        = "images/xp/Tminus.png";
            webFXTreeConfig.tPlusIcon        = "images/xp/Tplus.png";
            webFXTreeConfig.iIcon            = "images/xp/I.png";
            webFXTreeConfig.lIcon            = "images/xp/L.png";
            webFXTreeConfig.tIcon            = "images/xp/T.png";
            var rti;
            var tree = new WebFXTree("行政区划");
            tree.add(new WebFXLoadTreeItem("安徽省", "GetCity"));
            document.write(tree);
        </script>
    </body>
</html>
{% endcodeblock%}

2. Servlet代码

{% codeblock Servlet lang:java %}
/*
 * To change this template, choose Tools | Templates
 * and open the template in the editor.
 */

package org.database.common;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.database.model.Area;
import org.database.treeList.AreaList;

/**
 *
 * @author 木易有峰
 */
public class GetCity extends HttpServlet {
   
    /** 
     * Processes requests for both HTTP <code>GET</code> and <code>POST</code> methods.
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    protected void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {
        response.setContentType("text/xml;charset=UTF-8");
        PrintWriter out = response.getWriter();
        try {
            out.println("<tree>");
            AreaList menus = new AreaList();
            List list = menus.getCityAreas();
            Area area;
           for (int i = 0; i < list.size(); i++) {
                area = (Area) list.get(i);
                out.println("<tree text='" + area.getQymc() + "'>");
                if(!area.equals(null)){
                    List alist = menus.getCountAreas(area.getQybm().substring(0,4));
                    for(int j = 0; j < alist.size(); j ++){
                        Area a = (Area)list.get(j);
                        out.println("<tree text='" + a.getQymc() + "' />");
                    }
                }
                out.print("</tree>");
            }
            out.println("</tree>");
        } finally { 
            out.close();
        }
    } 

    // <editor-fold defaultstate="collapsed" desc="HttpServlet methods. Click on the + sign on the left to edit the code.">
    /** 
     * Handles the HTTP <code>GET</code> method.
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {
        processRequest(request, response);
    } 

    /** 
     * Handles the HTTP <code>POST</code> method.
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {
        processRequest(request, response);
    }

    /** 
     * Returns a short description of the servlet.
     * @return a String containing servlet description
     */
    @Override
    public String getServletInfo() {
        return "Short description";
    }// </editor-fold>

}
{% endcodeblock %}

3. WEB配置

{% codeblock web.xml lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_4.xsd">
    <servlet>
        <servlet-name>GetCity</servlet-name>
        <servlet-class>org.database.common.GetCity</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>GetCity</servlet-name>
        <url-pattern>/GetCity</url-pattern>
    </servlet-mapping>
    <session-config>
        <session-timeout>
            30
        </session-timeout>
    </session-config>
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
        </welcome-file-list>
    </web-app>
{% endcodeblock %}
