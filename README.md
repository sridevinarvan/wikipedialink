import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;

import java.util.HashSet;
import java.util.Set;

public class WikipediaLinkScraper {

    public static boolean isValidWikiLink(String link) {
        // Validate if the link is a valid Wikipedia link
        return link.matches("^https?://en\\.wikipedia\\.org/wiki/[\\w()]+$");
    }

    public static Set<String> getWikiLinks(WebDriver driver) {
        // Get the unique Wikipedia links from the current page
        Set<String> wikiLinks = new HashSet<>();
        for (WebElement link : driver.findElements(By.cssSelector("a[href^='/wiki/']"))) {
            String href = link.getAttribute("href");
            if (isValidWikiLink(href)) {
                wikiLinks.add(href);
            }
        }
        return wikiLinks;
    }

    public static void scrapeWikipediaLinks(String startUrl, int n) {
        if (!isValidWikiLink(startUrl)) {
            throw new IllegalArgumentException("Invalid Wikipedia link!");
        }

        System.setProperty("webdriver.chrome.driver", "path_to_chromedriver"); // Set the path to chromedriver

        WebDriver driver = new ChromeDriver();
        driver.get(startUrl);

        Set<String> visitedLinks = new HashSet<>();
        Set<String> linksToVisit = new HashSet<>();
        linksToVisit.add(startUrl);

        for (int i = 0; i < n; i++) {
            Set<String> newLinks = new HashSet<>();

            for (String currentUrl : linksToVisit) {
                if (!visitedLinks.contains(currentUrl)) {
                    visitedLinks.add(currentUrl);
                    System.out.println("Cycle " + (i + 1) + ": Visiting " + currentUrl);
                    driver.get(currentUrl);
                    newLinks.addAll(getWikiLinks(driver));
                }
            }

            linksToVisit = newLinks;
        }

        driver.quit();
    }

    public static void main(String[] args) {
        try {
            String wikiLink = "https://en.wikipedia.org/wiki/Main_Page"; // Replace with user input if desired
            int nCycles = 3; // Replace with user input if desired

            if (1 <= nCycles && nCycles <= 3) {
                scrapeWikipediaLinks(wikiLink, nCycles);
            } else {
                System.err.println("Please enter a valid integer between 1 and 3.");
            }
        } catch (IllegalArgumentException iae) {
            System.err.println(iae.getMessage());
        }
    }
}
