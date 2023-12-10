---
layout: post
title: Flavour of Async/await ES2017
---

One really exciting thing for JS Developer that last week was dropped Node 7.6 which brought support for `async/await`. If you haven't heard of async/await before, it's an amazing update to JavaScript (ES2017) that allows us to write synchronous looking asynchronous code without nested callbacks or even chained `.then()` promises.

```
/* Forgot Password Flow*/
export default forgotPassword = async(req,res,next) => {
  // Checking if user exist
  const user = await User.findOne({email : req.body.email}) ;

     if(!user){
       req.flash = ('error' , 'No Account found with this email');
       return res.redirect('/login')
     }
// Set password reset token for user
  user.resetPasswordToken = crypto.randomBytes(20).toString('hex');
  user.resetPasswordExpires = Date.now() + 3600000 ; // 1 hour
  await user.save()
// Sent Password reset email to User
  await mail.send({
     user,
     file:'password-reset',
     resetUrl : `http://${req.headers.host}/account/reset/${user.resetPasswordToken}`,
     subject: 'Password Reset'
   });
req.flash('success', 'You have been emailed a password reset link.');
res.redirect('/login');
};
```
Happy coding :)