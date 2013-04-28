## Every Project Needs a Home

<p><img src="https://raw.github.com/OlegIlyenko/my-blog-images/master/hacking-scala/scaldi-pamflet-b.png" alt="image" width="680" style="border: none" /></p>
<p>Recently I created home page for my project <a href="http://olegilyenko.github.com/scaldi/Scaldi.html" target="_blank">Scaldi</a>. I wanted to make it for a long time, but from the other hand I don't want to spend much time finding some hosting and maintaining its infrastructure, making page design, etc. <a href="https://github.com/OlegIlyenko/scaldi" target="_blank">Github page</a>&nbsp;is nice, but still I would like to have somethng more simple and unique as a project's front page.&nbsp;So my main goals were:</p>
<ul>
<li>Use very nice feature of Github -&nbsp;<a href="http://pages.github.com/" target="_blank">gh-pages</a>&nbsp;to host my pages</li>
<li>Use <a href="http://daringfireball.net/projects/markdown/" target="_blank">markdown</a> to create my pages without spending much time making page design</li>
<li>Design that comes out-of-box should be clean and pretty</li>
<li>Integrate pages generation and publishing in SBT build process</li>
</ul>
<!-- more -->
<p><strong><span style="font-size: x-large;">Pamflet</span></strong></p>
<p>And I finally found solution that addresses all these requirements. <a href="http://pamflet.databinder.net/Pamflet.html" target="_blank">Pamflet</a> is very nice Scala library that allows you to define your site using <em>markdown</em>. It's very easy to use: you just create your pages that are named according to their position in the resulting table of contents with <strong>.md</strong> or <strong>.markdown</strong> extension. You can also define some properties in the <strong>template.properties</strong>&nbsp;which you can use in your pages (they would be processed using <a href="http://www.stringtemplate.org/" target="_blank">StringTemplate</a>). If you want, you can add some custom CSS in order to make it even more prettier. So resulting file structure should look like this:</p>
<ul>
<li>00.md</li>
<li>01.md</li>
<li>02.md</li>
<li>custom.css</li>
<li>template.properties</li>
</ul>
<p><a href="http://pamflet.databinder.net/Pamflet.html" target="_blank">Pamflet</a>&nbsp;has even more features, but it was enough for me to start creating&nbsp;<a href="http://olegilyenko.github.com/scaldi/Scaldi.html" target="_blank">Scaldi</a>'s home page. So I ended up <a href="https://github.com/OlegIlyenko/scaldi/tree/master/docs" target="_blank">with following files in my project's <strong>docs</strong>&nbsp;folder</a>. It was pretty straightforward to create, but now I need to generate actual HTML from my <em>markdown </em>markup.</p>
<p><strong><span style="font-size: x-large;">SBT Integration</span></strong></p>
<p>In order to generate my pages and publish them to the <strong>gh-pages </strong>I need following SBT plugins:</p>
<ul>
<li><a href="https://github.com/n8han/pamflet-plugin" target="_blank">pamflet-plugin</a> - it generates actual site</li>
<li><a href="https://github.com/jsuereth/xsbt-ghpages-plugin" target="_blank">xsbt-ghpages-plugin</a> - adds generated pages to the <em>gh-pages</em> branch and pushes changes to github</li>
</ul>
<p>At first you need to create&nbsp;&nbsp;<em>gh-pages</em>&nbsp;branch in your git repository. You can make it like this:</p>
<p>
<script src="https://gist.github.com/OlegIlyenko/1462353.js"></script>
</p>
<p>After executing this, you need to add <em>SBT</em> plugins in your&nbsp;<strong>project/project/Build.scala</strong>:</p>
<p>
	<script src="https://gist.github.com/OlegIlyenko/1462125.js"></script>
</p>
<p>After adding these plugins, you should be able to generate HTML pages. All you need to do is to execute&nbsp;<strong>write-pamflet </strong>task and your pages would be generated in <strong>target/docs </strong>folder.</p>
<p>Now comes your main <strong>build.sbt </strong>file:</p>
<p>
	<script src="https://gist.github.com/OlegIlyenko/1462166.js"></script>
</p>
<p>I hope comments will help you in understanding what each of these settings does. So now in order to publish my pages I need to execute&nbsp;<strong>ghpages-push-site </strong>task. I can also run&nbsp;<strong>ghpages-synch-local </strong>in order to add pages to the <em>gh-pages</em><strong> </strong>branch locally.</p>
<p>As bonus, Scalasodocs would be also published by&nbsp;<a href="https://github.com/jsuereth/xsbt-ghpages-plugin" target="_blank">xsbt-ghpages-plugin</a>.</p>
<h1><span style="font-size: large;"><strong>Conclusion</strong></span></h1>
<p>It was pretty easy to integrate&nbsp;<a href="http://pamflet.databinder.net/Pamflet.html" target="_blank"><strong>Pamflet</strong></a>&nbsp;with <strong><a href="https://github.com/harrah/xsbt" target="_blank">SBT</a></strong> and&nbsp;<a href="http://pages.github.com/" target="_blank"><strong>gh-pages</strong></a>. I ended up with <a href="http://olegilyenko.github.com/scaldi/Scaldi.html" target="_blank">nice looking site</a> and published Scaladoc. And it's really easy to manage and publish them.</p>
<p>I&nbsp;hope this post will also help <em>you </em>to setup your project's home page in a matter of minutes, because, you know, <em>every project needs a home</em> :)</p>