2025-05-23 18:12

Status:

Tags: [[eWPT]]  [[Web Fingerprinting and Enumeration]] #Information_Gathering
###### Prerequisites:

[[WAF Detection]]

# Crawling&Spidering

---

## Website Crawling & Spidering

Understanding the content and structure of a website is a key part of web enumeration. Crawling and Spidering are two techniques used for this purpose.

### Crawling (Passive)

As described in the "Passive Crawling & Spidering With Burp Suite & OWASP ZAP" section (Page 37) and detailed on Page 38:

*   **Definition:** Crawling is the process of navigating around the web application. This involves:
    *   Following links.
    *   Submitting forms (where possible and relevant to passive discovery).
    *   Logging in (where possible, e.g., with provided test credentials, still often considered passive if just browsing authorized areas).
*   **Objective:** To map out and catalog the web application and the navigational paths within it.
*   **Nature:** Typically considered a **passive** activity.
    *   Engagement with the target is done via what is publicly accessible.
*   **Tools Example:** Burp Suite's passive crawler.
    *   Can be utilized to help map out the web application.
    *   Aids in better understanding how the application is set up and how it works.
*   **Learning Objective:** You will learn how to perform spidering and crawling to identify the content structure of websites. (Page 5, Page 58)

### Spidering (Often Active)

As detailed on Page 39, often discussed alongside crawling:

*   **Definition:** Spidering is the process of automatically discovering new resources (URLs) on a web application or site.
*   **Process:**
    *   It typically begins with a list of target URLs called "seeds."
    *   The Spider visits these URLs, identifies hyperlinks in the page content.
    *   Adds these newly found hyperlinks to its list of URLs to visit.
    *   Repeats this process recursively.
*   **Nature:**
    *   Spidering can be quite "loud" (i.e., generate noticeable traffic to the target).
    *   As a result, it is typically considered to be an **active** information gathering technique.
*   **Tools Example:** OWASP ZAP's Spider.
    *   Can be utilized to automate the process of spidering a web application.
    *   Helps to map out the web application.
    *   Allows learning more about how the site is laid out and how it works.
*   **Learning Objective:** You will learn how to perform spidering and crawling to identify the content structure of websites. (Page 5, Page 58)

**Key Distinction often implied:**
*   **Crawling** can be more manual or involve passively observing traffic (e.g., browsing while proxying through Burp Suite which maps site structure based on your requests).
*   **Spidering** is usually an automated process specifically designed to recursively find all linked resources.

Using OWASP ZAP
