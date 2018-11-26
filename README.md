why
---

```diff
-        let end64 : i64 = i64::from( user.end );
-        let u_end = NaiveDateTime::from_timestamp(end64, 0);
-        let d_end = DateTime::<Utc>::from_utc(u_end, Utc);
-        let duration = d_end.signed_duration_since(Local::now());
+        letrec!( end64    = i64::from( user.end ) : i64
+               , u_end    = NaiveDateTime::from_timestamp(end64, 0)
+               , d_end    = DateTime::<Utc>::from_utc(u_end, Utc)
+               , duration = d_end.signed_duration_since(Local::now()) );
```

```diff
 pub fn activate(license_key : &String) -> String {
-  let provided = CString::new(license_key.as_str()).unwrap();
-  let provided_ptr = provided.as_ptr();
-  let end = Local::now().checked_add_signed(Duration::days(30 * 3));
-  let dtEnd = end.unwrap().timestamp();
-  let result = activate_licensing(provided_ptr, dtEnd, true, false, 7);
-  let c_str: &CStr = unsafe { CStr::from_ptr(result) };
-  let str_slice: &str = c_str.to_str().unwrap();
-  let str_buf: String = str_slice.to_owned();
-  str_buf
+  letrec!
+    ( provided      = CString::new(license_key.as_str()).unwrap()
+    , provided_ptr  = provided.as_ptr()
+    , end           = Local::now().checked_add_signed(Duration::days(30 * 3))
+    , dtEnd         = end.unwrap().timestamp()
+    , result        = activate_licensing(provided_ptr, dtEnd, true, false, 7)
+    , c_str         = unsafe { CStr::from_ptr(result) } : &CStr
+    , str_slice     = c_str.to_str().unwrap() : &str );
+
+  str_slice.to_owned() : String
 }
```

notes
-----

 - type specifiers on lhs will not work, I'd like it to be moved to rhs so please use `#![feature(type_ascription)]` possible way to have it done is `$(: $t:ty)*` after `$lhs:ident`
 - using macros for stylistic changes is not welcomed usually
 - not very good in debugging (but not fully sure)

code
----

```rust
#[macro_export]
macro_rules! letrec {
  ($init:ident = $val:expr, $($lhs:ident = $rhs:expr),*) => {
      let $init = $val;
    $(
      let $lhs = $rhs;
    )*
  };
}
```

how to
------

put following in `lib.rs` or `main.rs`

```rust
#![feature(type_ascription)]
#[macro_use] pub mod letrec;
```


Thanks to Aurorans Solis for helping with writing it.
