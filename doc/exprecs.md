Module exprecs
==============


<h1>Module exprecs</h1>

* [Description](#description)
* [Data Types](#types)
* [Function Index](#index)
* [Function Details](#functions)


Parse transform for generating record access functions.



__Authors:__ : Ulf Wiger ([`ulf.wiger@ericsson.com`](mailto:ulf.wiger@ericsson.com)).

<h2><a name="description">Description</a></h2>

  

This parse transform can be used to reduce compile-time
dependencies in large systems.


In the old days, before records, Erlang programmers often wrote
access functions for tuple data. This was tedious and error-prone.
The record syntax made this easier, but since records were implemented
fully in the pre-processor, a nasty compile-time dependency was
introduced.


This module automates the generation of access functions for
records. While this method cannot fully replace the utility of
pattern matching, it does allow a fair bit of functionality on
records without the need for compile-time dependencies.


Whenever record definitions need to be exported from a module,
inserting a compiler attribute,
`export_records([RecName|...])` causes this transform
to lay out access functions for the exported records:

<pre>
  -module(test_exprecs).
 
  -record(r, {a, b, c}).
  -export_records([r]).
 
  -export(['#new-'/1, '#info-'/1, '#info-'/2, '#pos-'/2,
           '#is_record-'/2, '#get-'/2, '#set-'/2, '#fromlist-'/2,
           '#new-r'/0, '#new-r'/1, '#get-r'/2, '#set-r'/2,
           '#pos-r'/1, '#fromlist-r'/2, '#info-r'/1]).
 
  '#new-'(r) -> '#new-r'().
 
  '#info-'(RecName) -> '#info-'(RecName, fields).
 
  '#info-'(r, Info) -> '#info-r'(Info).
 
  '#pos-'(r, Attr) -> '#pos-r'(Attr).
 
  '#is_record-'(r, Rec)
      when tuple_size(Rec) == 3, element(1, Rec) == r ->
      true;
  '#is_record-'(_, _) -> false.
 
  '#get-'(Attrs, Rec) when is_record(Rec, r) ->
      '#get-r'(Attrs, Rec).
 
  '#set-'(Vals, Rec) when is_record(Rec, r) ->
      '#set-r'(Vals, Rec).
 
  '#fromlist-'(Vals, Rec) when is_record(Rec, r) ->
      '#fromlist-r'(Vals, Rec).
 
  '#new-r'() -> #r{}.
 
  '#new-r'(Vals) -> '#set-r'(Vals, #r{}).
 
  '#get-r'(Attrs, R) when is_list(Attrs) ->
      ['#get-r'(A, R) || A <- Attrs];
  '#get-r'(a, R) -> R#r.a;
  '#get-r'(b, R) -> R#r.b;
  '#get-r'(c, R) -> R#r.c;
  '#get-r'(Attr, R) ->
      erlang:error(bad_record_op, ['#get-r', Attr, R]).
 
  '#set-r'(Vals, Rec) ->
      F = fun ([], R, _F1) -> R;
              ([{a, V} | T], R, F1) -> F1(T, R#r{a = V}, F1);
              ([{b, V} | T], R, F1) -> F1(T, R#r{b = V}, F1);
              ([{c, V} | T], R, F1) -> F1(T, R#r{c = V}, F1);
              (Vs, R, _) ->
                  erlang:error(bad_record_op, ['#set-r', Vs, R])
          end,
      F(Vals, Rec, F).
 
  '#fromlist-r'(Vals, Rec) ->
      AttrNames = [{a, 2}, {b, 3}, {c, 4}],
      F = fun ([], R, _F1) -> R;
              ([{H, Pos} | T], R, F1) ->
                  case lists:keyfind(H, 1, Vals) of
                    false -> F1(T, R, F1);
                    {_, Val} -> F1(T, setelement(Pos, R, Val), F1)
                  end
          end,
      F(AttrNames, Rec, F).
 
  '#pos-r'(a) -> 2;
  '#pos-r'(b) -> 3;
  '#pos-r'(c) -> 4;
  '#pos-r'(_) -> 0.
 
  '#info-r'(fields) -> record_info(fields, r);
  '#info-r'(size) -> record_info(size, r).
  </pre>


<h2><a name="types">Data Types</a></h2>





<h3 class="typedecl"><a name="type-form">form()</a></h3>




<pre>form() = any()</pre>



<h3 class="typedecl"><a name="type-forms">forms()</a></h3>




<pre>forms() = [<a href="#type-form">form()</a>]</pre>



<h3 class="typedecl"><a name="type-options">options()</a></h3>




<pre>options() = [{atom(), any()}]</pre>


<h2><a name="index">Function Index</a></h2>



<table width="100%" border="1" cellspacing="0" cellpadding="2" summary="function index"><tr><td valign="top"><a href="#parse_transform-2">parse_transform/2</a></td><td></td></tr></table>




<h2><a name="functions">Function Details</a></h2>


<a name="parse_transform-2"></a>

<h3>parse_transform/2</h3>





<pre>parse_transform(Forms::<a href="#type-forms">forms()</a>, Options::<a href="#type-options">options()</a>) -> <a href="#type-forms">forms()</a></pre>
<br></br>


