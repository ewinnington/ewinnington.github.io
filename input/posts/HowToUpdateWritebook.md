Title: Adding LaTeX Maths to Writebook 
Published: 13/7/2024 15:00
Tags: [markdown, Writebook] 
---

# How to add LaTeX maths to Writebook 

I've started using ONCE [Writebook](https://once.com/writebook) to host some markdown documents, including my book on Optimization applied to energy products. But that book contains tons of Markdown LaTeX formatted mathematics, which Writebook does not support at this time.

So I patched support for it into the docker image.

## Copy out the app layout file

Assuming your docker image is named writebook. 

```
sudo docker cp writebook:/rails/app/views/layouts/application.html.erb .
```

## Modify the application erb to add support 

In the Head Section add

```
 <script type="text/javascript">
      window.MathJax = {
        tex: {
          inlineMath: [['$', '$']],
          displayMath: [['$$', '$$']],
          processEscapes: true
        }
      };
    </script>
      
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.2/es5/tex-mml-chtml.js"></script>
```

Thing is, it doesn't render the first time round, so I have to refresh the page to render, but it's not too much of an issue right now, I actually like to see the LaTeX maths code before I see the rendered version. 

<!--
Not working right now ! 

## How to add a render on end of page load to ensure it typesets first time

Above the ```</Body>``` tag at the bottom of the page, add
```
    <script type="text/javascript">
    document.addEventListener("DOMContentLoaded", function() {
      MathJax.typeset();
    });
  </script>
```

-->

## Save and copy the file back 

Then finally copy the file back into the docker image and restart the image

```
sudo docker cp application.html.erb writebook:/rails/app/views/layouts/application.html.erb
sudo docker restart writebook
```

