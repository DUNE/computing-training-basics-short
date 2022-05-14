---
title: Quiz on Best Programming Practices
teaching: 15
exercises: 0
questions:
- Is programming your forte?
objectives:
- Validate your understanding by working through some examples.
keypoints:
- Practice makes perfect.
---

## Video Session

<!--The session will be captured on video a placed here after the workshop for asynchronous study.-->
The session was captured for your asynchronous review.

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/qENjAa0L1lU" title="DUNE Computing Tutorial May 2022 Quiz Programming Efficiency" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

## Quiz Time

### Question 01

In the following two code snippets, which example is more efficient, A or B?

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;"> <!--HEADER-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (A)</div><!--BAD-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071">Code Example (B)</div><!--GOOD-->


<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double sum = 0;<br>
double f = TMath::Sqrt(0.5);<br>
for (size_t i=0; i<n_channels; ++i)<br>
{<br>
  sum += result.at(i)*f;<br>
}<br>
</div><!--BAD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double sum = 0;<br>
for (size_t i=0; i<n_channels; ++i)<br>
{<br>
  sum += result.at(i)/TMath::Sqrt(2.0);<br>
}<br>
</div><!--GOOD-->

</div><!--side by side table by DeMuth-->


### Question 02

In the following two code snippets, which example is more efficient, A or B?


<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;"><!--HEADER-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (A)</div><!--BAD-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (B)</div> <!--GOOD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double r = TMath::Sqrt( x*x + y*y );
</div><!--BAD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double r = TMath::Power(  TMath::Power(x,2) + TMath::Power(y,2), 0.5);
</div><!--GOOD-->

</div><!--side by side table by DeMuth-->


### Question 03

In the following two code snippets, which example is more efficient, A or B?

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (A)</div><!--GOOD-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (B)</div>  <!--BAD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double r = TMath::Sqrt( slow_function_calculating_x()*<br>slow_function_calculating_x() + <br> slow_function_calculating_y()* <br> slow_function_calculating_y()  );
</div><!--GOOD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double r = TMath::Sqrt( TMath::Sq(slow_function_calculating_x()) + <br> TMath::Sq(slow_function_calculating_y()));
</div><!--BAD-->

</div><!--side by side table by DeMuth-->

<!-- **Don't call `sqrt()` if you don’t have to.** -->

### Question 04

In the following two code snippets, which example is more efficient, A or B?

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;"><!--HEADER-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (A)</div><!--GOOD-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (B)</div><!--BAD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
if (TMath::Sqrt( x*x + y*y ) < rcut )<br>
{<br>
  do_something();<br>
}
</div><!--GOOD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double rcutsq = rcut*rcut;<br>
if (x*x + y*y < rcutsq)<br>
{<br>
  do_something();<br>
}
</div><!--BAD-->

</div><!--side by side table by DeMuth-->


### Question 05

In the following two code snippets, which example is more efficient, A or B?

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;"><!--HEADER-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (A)</div><!--BAD-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (B)</div><!--GOOD-->


<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double sum = 0;<br>
std::vector &lt;double&gt; results;<br>
(fill lots of results)<br>
for (size_t i=0; i<results.size(); ++i)<br>
{<br>
  sum += TMath::Sq(results.at(i));<br>
}
</div><!--BAD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double sum = 0;<br>
std::vector &lt;double&gt; results;<br>
(fill lots of results)<br>
for (size_t i=0; i<results.size(); ++i)<br>
{<br>
  float rsq = results.at(i)*result.at(i);<br>
  sum += rsq;<br>
}
</div><!--GOOD-->

</div><!--side by side table by DeMuth-->

<!-- **Minimize conversions between int and float or double** -->

### Question 06

In the following two code snippets, which example is more efficient, A or B?

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (A)</div><!--GOOD-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (B)</div><!--BAD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
int dune::VDColdboxChannelMapService:: getOfflChanFromWIBConnectorInfo (int wib, int wibconnector, int cechan)<br>
{<br>
  int r = -1;<br>
  auto fm1 = infotochanmap.find(wib);<br>
  if (fm1 == infotochanmap.end()) return r;<br>
  auto m1 = fm1-&gt;second;<br>
  auto fm2 = m1.find(wibconnector);<br>
  if (fm2 == m1.end()) return r;<br>
  auto m2 = fm2-&gt;second;<br>
  auto fm3 = m2.find(cechan);<br>
  if (fm3 == m2.end()) return r;<br>
  r = fm3->second;  <br>
  return r;<br>
</div><!--GOOD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
int dune::VDColdboxChannelMapService:: getOfflChaFromWIBConnectorInfo (int wib, int wibconnector, int cechan)<br>
{<br>
  int r = -1;<br>
  auto fm1 = infotochanmap.find(wib);<br>
  if (fm1 == infotochanmap.end()) return r;<br>
  auto& m1 = fm1-&gt;second;<br>
  auto fm2 = m1.find(wibconnector);<br>
  if (fm2 == m1.end()) return r;<br>
  auto& m2 = fm2-&gt;second;<br>
  auto fm3 = m2.find(cechan);<br>
  if (fm3 == m2.end()) return r;<br>
  r = fm3->second;  <br>
  return r;<br>
}<br>
</div><!--BAD-->

</div><!--side by side table by DeMuth-->

### Question 07

<!-- In the following two code snippets, which example is more efficient, A or B?-->

Can you point out the errors in the following code blocks?


<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;"><!--HEADER-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (A)</div><!--BAD-->

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (B)</div> <!--GOOD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">

#include  &#60;iostream&#62;  <br>
int main(int argc, char &#8727;&#8727;argv)<br>
{<br>
&nbsp;&nbsp;for (int i=0; i&#60; 10000; ++i)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{ <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int  &#8727;j = new int[1000]<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} <br><br>
&nbsp;&nbsp;int kt;<br>
&nbsp;&nbsp;std::cout &#60;&#60; kt &#60;&#60; std::endl;<br><br>

&nbsp;&nbsp;int p[100];<br>
&nbsp;&nbsp;p[100] = 5;<br>
&nbsp;&nbsp;std::cout &#60;&#60; p[100] &#60;&#60; std::endl;<br><br>
&nbsp;&nbsp;int &#8727;k;<br>
&nbsp;&nbsp;k= 0;<br>
&nbsp;&nbsp;&#8727;k = 6;<br>
&nbsp;&nbsp;std::cout &#60;&#60; &#8727;k &#60;&#60; std::endl;<br>
}<br>

</div><!--BAD-->

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">

#include  &#60;iostream&#62;  <br>
int main(int argc, char &#8727;&#8727;argv)<br>
{<br>
&nbsp;&nbsp;for (int i=0; i&#60; 10000; ++i)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{ <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int  &#8727;j = new int[1000]<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} <br><br>
&nbsp;&nbsp;int kt= 0;<br>
&nbsp;&nbsp;std::cout &#60;&#60; kt &#60;&#60; std::endl;<br><br>

&nbsp;&nbsp;int p[100];<br>
&nbsp;&nbsp;p[99] = 5;<br>
&nbsp;&nbsp;std::cout &#60;&#60; p[99] &#60;&#60; std::endl;<br><br>
&nbsp;&nbsp;int &#8727;k=k;<br>
&nbsp;&nbsp;k= 0;<br>
&nbsp;&nbsp;&#8727;k = 6;<br>
&nbsp;&nbsp;std::cout &#60;&#60; &#8727;k &#60;&#60; std::endl;<br>
}<br>

</div><!--GOOD-->

</div><!--side by side table by DeMuth-->


<!-- The session will be captured on video a placed here after the workshop for asynchronous study.-->

<!--<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/E8n0saHdr4E" title="DUNE Computing Tutorial May 2021 Day 3 Quiz on Best Programming Practices" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>-->

[indico-timetable]: https://indico.fnal.gov/event/48756/timetable/#all
[sc-etherpad]: https://pad.carpentries.org/


{%include links.md%} 

