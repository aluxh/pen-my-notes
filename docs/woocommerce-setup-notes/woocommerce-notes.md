# WooCommerce Setup Notes

- [WooCommerce Setup Notes](#woocommerce-setup-notes)
  - [Set Up Automated Order Notification Emails with Mailchimp](#set-up-automated-order-notification-emails-with-mailchimp)
  - [Set Up Pop-up Dialog with Mailchimp in Wordpress to encourage sign-up and receive discount codes](#set-up-pop-up-dialog-with-mailchimp-in-wordpress-to-encourage-sign-up-and-receive-discount-codes)
  - [Create Discount Code In WooCommerce](#create-discount-code-in-woocommerce)

## Set Up Automated Order Notification Emails with Mailchimp

To begin, you need to have a Mailchimp account. Then, from the Wordpress/WooCommerce, install the plugin - [Mailchimp for WooCommerce](https://wordpress.org/plugins/mailchimp-for-woocommerce/) (install the latest version).

1. Log into *Mailchimp*.
2. Select **Automate** > **Email**.
3. Go to *E-Commerce* tab, and select **Enable Order Notifications**.
4. Choose the type of emails you wish to automate, and design them.
5. After that, select continue and start the automation.
6. Log into *Wordpress*, go to *WooCommerce* section.
7. Go to *Settings* > *Emails* tab.
8. Disable those emails that you're automating in WooCommerce. In my case, Processing order, Complete order, Cancel order and etc..

**Note:** One challenge with Mailchimp's automated emails is you're unable to add shipment tracking code to the email automatically. Another enhancement that I'd would love to see is adding some personalization in email, as well as changing the text of the button in the email.

## Set Up Pop-up Dialog with Mailchimp in Wordpress to encourage sign-up and receive discount codes

To begin, you need to have a Mailchimp account, and also an audience list.

1. Log into *Mailchimp*.
2. Select **Audience** from the top navigation bar.
3. Select the **Manage Audience** drop-down menu. And, choose **Signup forms**.
4. Select **Subscriber Pop-up**.
5. Then, follow the wizard to create your email design. If you wish to include a discount code, you should [create the code](#Create-discount-code-in-WooCommerce) in *WooCommerce*, and add it to the email.
6. Under the *Design* > *Pop-up settings*, I usually set *Display* pop-up **After 20 seconds**.
7. With API 3.0, you can follow the wizard to the end, and it will automatically enabled to run.

## Create Discount Code In WooCommerce

1. Log into *Wordpress*, go to *WooCommerce* section.
2. Go to **Coupons**. Then, follow the wizard to create your discount code.
3. Additional Notes:
   1. Under *General* tab, 
      1. you can set discount type to be in *percentage* or *fixed amount*.
      2. Then, you can set the amount in *Coupon Amount*. You don't need to include percentage or dollar sign.
      3. Set an expiry date for the discount in *Coupon Expiry Date*. For new subscriber discount, you can leave it blank.
   2. Under *Usage Restriction* tab,
      1. It is important to **check** the checkbox - **Individual Use only**. This ensure that the code cannot be used together with other discount codes.
      2. If you want to restrict the discount code to be for those subscribers, you can put those emails in **Allowed Emails**.
   3. Under *Usage Limits* tab,
      1. You can set the number of discount code to be used. For example, you can issue the discount code to be used only 100 times, then it will disabled.
      2. You can set how many times each user can use the discount code. This is tied to his/her email address. For example, welcome discount should only be used once per subscriber.

## Shipment Tracking plugin for WooCommerce

You need to pay a high fee to use WooCommerce Shipment Tracking plugin. Therefore, I recommend that you use *Aftership* plugin instead. After installing, you should enable the tracking from the **Aftership** > **settings**. Thereafter, you will be able to insert tracking code into the orders.

Your customers will be able to check the tracking code from the Account page in Wordpress/WooCommerce. Unfortunately, the tracking code and link will not appear in the order email. This makes it inconvenient for the customers to track their order.
