<!DOCTYPE html>
<!-- saved from url=(0065)https://documentation.wazuh.com/2.0/installation-guide/index.html -->
<html class=" js flexbox canvas canvastext webgl no-touch geolocation postmessage websqldatabase indexeddb hashchange history draganddrop websockets rgba hsla multiplebgs backgroundsize borderimage borderradius boxshadow textshadow opacity cssanimations csscolumns cssgradients cssreflections csstransforms csstransforms3d csstransitions fontface no-generatedcontent video audio localstorage sessionstorage webworkers applicationcache svg inlinesvg smil svgclippaths" lang="en" style=""><!--<![endif]--><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  
  
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  
  <title>Installation guide — Documentation 2.0 documentation</title>
  

  

  <script async="" src="./use-case_files/analytics.js.tải xuống"></script><script>
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

    ga('create', 'UA-65317123-3', 'auto');
    ga('send', 'pageview');

  </script>

  
  
    <link rel="shortcut icon" href="https://documentation.wazuh.com/2.0/_static/favicon.ico">
  

  

  
  
    

  

  
  
    <link rel="stylesheet" href="./use-case_files/theme.css" type="text/css">
  

  
    <link rel="stylesheet" href="./use-case_files/lightbox.css" type="text/css">
  
    <link rel="stylesheet" href="./use-case_files/my-styles.css" type="text/css">
  

  
        <link rel="index" title="Index" href="https://documentation.wazuh.com/2.0/genindex.html">
        <link rel="search" title="Search" href="https://documentation.wazuh.com/2.0/search.html">
    <link rel="top" title="Documentation 2.0 documentation" href="https://documentation.wazuh.com/2.0/index.html">
        <link rel="next" title="Installing Wazuh server" href="https://documentation.wazuh.com/2.0/installation-guide/installing-wazuh-server/index.html">
        <link rel="prev" title="Use cases" href="https://documentation.wazuh.com/2.0/getting-started/use-cases.html"> 

  
  <script src="./use-case_files/modernizr.min.js.tải xuống"></script>

</head>

<body class="wy-body-for-nav" role="document">

  <div class="wy-grid-for-nav">

    
    <nav data-toggle="wy-nav-shift" class="wy-nav-side">
      <div class="wy-side-scroll">
        <div class="wy-side-nav-search">
          

          
            <a href="https://documentation.wazuh.com/2.0/index.html" class="icon icon-home"> Documentation
          

          
            
            <img src="./use-case_files/logo.png" class="logo">
          
          </a>

          
            
            
              <div class="version">
                 2.0
              </div>
            
          

          
<div role="search">
  <form id="rtd-search-form" class="wy-form" action="https://documentation.wazuh.com/2.0/search.html" method="get">
    <input type="text" name="q" placeholder="Search docs">
    <input type="hidden" name="check_keywords" value="yes">
    <input type="hidden" name="area" value="default">
  </form>
</div>

          
        </div>

        <div class="wy-menu wy-menu-vertical" data-spy="affix" role="navigation" aria-label="main navigation">
          
            
            
                <ul class="current">
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/getting-started/index.html">Getting started</a></li>
<li class="toctree-l1 current"><a class="current reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/index.html#"><span class="toctree-expand"></span>Installation guide</a><ul>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/installing-wazuh-server/index.html">Installing Wazuh server</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/installing-elastic-stack/index.html">Installing Elastic Stack</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/installing-wazuh-agent/index.html">Installing Wazuh agent</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/optional-configurations/index.html">Optional configurations</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/upgrading/index.html">Upgrading Wazuh</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/virtual-machine.html">Virtual Machine</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html">Packages List</a></li>
</ul>
</li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/user-manual/index.html">User manual</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/docker/index.html">Docker</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/deploying-with-puppet/index.html">Deploying with Puppet</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/pci-dss/index.html">Using Wazuh for PCI DSS</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/amazon/index.html">Using Wazuh for AWS</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/migrating-from-ossec/index.html">Migrating from OSSEC</a></li>
</ul>

            
          
        </div>
      </div>
    </nav>

    <section data-toggle="wy-nav-shift" class="wy-nav-content-wrap">

      
      <nav class="wy-nav-top" role="navigation" aria-label="top navigation">
        <i data-toggle="wy-nav-top" class="fa fa-bars"></i>
        <a href="https://documentation.wazuh.com/2.0/index.html">Documentation</a>
      </nav>


      
      <div class="wy-nav-content">
        <div class="rst-content">
          

 



<div role="navigation" aria-label="breadcrumbs navigation">
  <ul class="wy-breadcrumbs">
    <li><a href="https://documentation.wazuh.com/2.0/index.html">Docs</a> »</li>
      
    <li>Installation guide</li>
      <li class="wy-breadcrumbs-aside">
        
          
            <a href="https://github.com/wazuh/wazuh-documentation/blob/2.0/source/installation-guide/index.rst" class="fa fa-github"> Edit on GitHub</a>
          
        
      </li>
  </ul>
  <hr>
</div>
          <div role="main" class="document" itemscope="itemscope" itemtype="http://schema.org/Article">
           <div itemprop="articleBody">
            
  <div class="section" id="installation-guide">
<span id="id1"></span><h1>Installation guide<a class="headerlink" href="https://documentation.wazuh.com/2.0/installation-guide/index.html#installation-guide" title="Permalink to this headline">¶</a></h1>
<p>This document will guide you through the installation process. For interactive help, our <a class="reference external" href="https://groups.google.com/d/forum/wazuh">email forum</a> is available. You can subscribe by sending an email to <a class="reference external" href="mailto:wazuh%2Bsubscribe%40googlegroups.com">Wazuh subscribe</a>.</p>
<p>Wazuh installation involves two central components, the Wazuh server, and Elastic Stack. In addition, Wazuh agents will need to be deployed to the monitored hosts in your environment:</p>
<ul class="simple">
<li><strong>Wazuh server:</strong> Runs the Wazuh manager, API and Filebeat (only necessary in distributed architecture). Collects and analyzes data from deployed agents.</li>
</ul>
<ul class="simple">
<li><strong>Elastic Stack</strong>: Runs the Elasticsearch engine, Logstash server and Kibana (including the Wazuh App). It reads, parses, indexes, and stores alert data generated by the Wazuh server.</li>
</ul>
<ul class="simple">
<li><strong>Wazuh agent</strong>: Runs on the monitored host, collecting system log and configuration data, and detecting intrusions and anomalies. It talks with the Wazuh server, to which it forwards collected data for further analysis.</li>
</ul>
<p>Distributed architectures do run the Wazuh server and Elastic Stack cluster (one or more servers) on different hosts. On the other hand, single-host architectures have Wazuh server and Elastic Stack installed in the same system. This guide covers both installation options.</p>
<p>The diagrams below list the components that are run per host, both for single-host and distributed architectures.</p>
<p><strong>Single-host architecture</strong>:</p>
<a class="" data-lightbox="group-default" href="./use-case_files/installing_wazuh_singlehost2.png" title="Installing Wazuh server - single server architecture" data-title="Installing Wazuh server - single server architecture"><img src="./use-case_files/installing_wazuh_singlehost2.png" class="align-center" width="100%" height="auto" alt="">
                </a><p><strong>Distributed architecture</strong>:</p>
<a class="" data-lightbox="group-default" href="./use-case_files/installing_wazuh2.png" title="Installing Wazuh server - distributed architecture" data-title="Installing Wazuh server - distributed architecture"><img src="./use-case_files/installing_wazuh2.png" class="align-center" width="100%" height="auto" alt="">
                </a><div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">Before installing the components please confirm time synchronization service is configured and working on your servers. This is most commonly done with <strong>NTP</strong>.  More info for <a class="reference external" href="https://help.ubuntu.com/lts/serverguide/NTP.html">Debian/Ubuntu</a> and <a class="reference external" href="http://www.tecmint.com/install-ntp-server-in-centos/">CentOS/RHEL/Fedora</a>.</p>
</div>
<div class="topic">
<p class="topic-title first">Contents</p>
<div class="toctree-wrapper compound">
<ul>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/installing-wazuh-server/index.html">Installing Wazuh server</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/installing-elastic-stack/index.html">Installing Elastic Stack</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/installing-wazuh-agent/index.html">Installing Wazuh agent</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/optional-configurations/index.html">Optional configurations</a><ul>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/optional-configurations/elastic_ssl.html">Setting up SSL for Filebeat and Logstash</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/optional-configurations/kibana_ssl.html">Setting up SSL and authentication for Kibana</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/optional-configurations/securing-api.html">Securing the Wazuh API</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/optional-configurations/elastic-tuning.html">Elasticsearch tuning</a></li>
</ul>
</li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/upgrading/index.html">Upgrading Wazuh</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/virtual-machine.html">Virtual Machine</a></li>
<li class="toctree-l1"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html">Packages List</a><ul>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#windows">Windows</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#mac-os-x">Mac OS X</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#red-hat">Red Hat</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#centos">CentOS</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#fedora">Fedora</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#ubuntu">Ubuntu</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#debian">Debian</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#solaris">Solaris</a></li>
<li class="toctree-l2"><a class="reference internal" href="https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html#ova-wazuh-2-0-1-elk-5-5-0">OVA Wazuh 2.0.1 + ELK 5.5.0</a></li>
</ul>
</li>
</ul>
</div>
</div>
</div>


           </div>
          </div>
          <footer>
  
    <div class="rst-footer-buttons" role="navigation" aria-label="footer navigation">
      
        <a href="https://documentation.wazuh.com/2.0/installation-guide/installing-wazuh-server/index.html" class="btn btn-neutral float-right" title="Installing Wazuh server" accesskey="n">Next <span class="fa fa-arrow-circle-right"></span></a>
      
      
        <a href="https://documentation.wazuh.com/2.0/getting-started/use-cases.html" class="btn btn-neutral" title="Use cases" accesskey="p"><span class="fa fa-arrow-circle-left"></span> Previous</a>
      
    </div>
  

  <hr>

  <div role="contentinfo">
    <p>
        © Copyright 2017, Wazuh, Inc.

    </p>
  </div> 

</footer>

        </div>
      </div>

    </section>

  </div>
  


  

    <script type="text/javascript">
        var DOCUMENTATION_OPTIONS = {
            URL_ROOT:'../',
            VERSION:'2.0',
            COLLAPSE_INDEX:false,
            FILE_SUFFIX:'.html',
            HAS_SOURCE:  true,
	    SOURCELINK_SUFFIX: '.txt'
        };
    </script>
      <script type="text/javascript" src="./use-case_files/jquery.js.tải xuống"></script>
      <script type="text/javascript" src="./use-case_files/underscore.js.tải xuống"></script>
      <script type="text/javascript" src="./use-case_files/doctools.js.tải xuống"></script>
      <script type="text/javascript" src="./use-case_files/jquery-1.11.0.min.js.tải xuống"></script>
      <script type="text/javascript" src="./use-case_files/lightbox.min.js.tải xuống"></script>
      <script type="text/javascript" src="./use-case_files/jquery-noconflict.js.tải xuống"></script>

  

  
  
    <script type="text/javascript" src="./use-case_files/theme.js.tải xuống"></script>
  

  
  
  <script type="text/javascript">
      jQuery(function () {
          SphinxRtdTheme.StickyNav.enable();
      });
  </script>
   


<div id="lightboxOverlay" class="lightboxOverlay" style="display: none;"></div><div id="lightbox" class="lightbox" style="display: none;"><div class="lb-outerContainer"><div class="lb-container"><img class="lb-image" src="https://documentation.wazuh.com/2.0/installation-guide/index.html"><div class="lb-nav"><a class="lb-prev" href="https://documentation.wazuh.com/2.0/installation-guide/index.html"></a><a class="lb-next" href="https://documentation.wazuh.com/2.0/installation-guide/index.html"></a></div><div class="lb-loader"><a class="lb-cancel"></a></div></div></div><div class="lb-dataContainer"><div class="lb-data"><div class="lb-details"><span class="lb-caption"></span><span class="lb-number"></span></div><div class="lb-closeContainer"><a class="lb-close"></a></div></div></div></div></body></html>