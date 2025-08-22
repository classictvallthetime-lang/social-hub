[index.html](https://github.com/user-attachments/files/21942017/index.html)
import React, { useState, useEffect } from 'react';

// The main App component for the RSS feed reader.
const App = () => {
    // State to hold the fetched feed items.
    const [feedItems, setFeedItems] = useState([]);
    // State for the loading indicator.
    const [isLoading, setIsLoading] = useState(true);
    // State for error messages.
    const [error, setError] = useState(null);
    // State for the user-defined RSS feed URL.
    const [rssUrl, setRssUrl] = useState('https://www.theverge.com/rss/index.xml');

    // A utility function to parse an XML string into a DOM document.
    const parseXml = (xmlString) => {
        const parser = new DOMParser();
        return parser.parseFromString(xmlString, "text/xml");
    };

    // A function to handle the fetch and parsing of the RSS feed.
    const fetchFeed = async (url) => {
        setIsLoading(true);
        setError(null);
        try {
            // Using a CORS proxy to bypass cross-origin issues.
            const corsProxy = 'https://api.allorigins.win/raw?url=';
            const response = await fetch(`${corsProxy}${encodeURIComponent(url)}`);

            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }

            const xmlText = await response.text();
            const xmlDoc = parseXml(xmlText);

            // Select all 'item' elements from the parsed XML.
            const items = xmlDoc.querySelectorAll("item");
            const newItems = [];

            // Iterate through each 'item' and extract relevant data.
            items.forEach(item => {
                const title = item.querySelector("title")?.textContent || "No Title";
                const link = item.querySelector("link")?.textContent || "#";
                const description = item.querySelector("description")?.textContent || "No Description";
                const pubDate = item.querySelector("pubDate")?.textContent || "No Date";
                const creator = item.querySelector("creator")?.textContent || "Unknown Creator";

                newItems.push({ title, link, description, pubDate, creator });
            });

            setFeedItems(newItems);
        } catch (e) {
            console.error("Error fetching or parsing feed:", e);
            setError("Failed to load RSS feed. Please check the URL.");
        } finally {
            setIsLoading(false);
        }
    };

    // useEffect hook to fetch the feed on initial component load.
    useEffect(() => {
        fetchFeed(rssUrl);
    }, [rssUrl]);

    // Handle form submission to update the URL and fetch a new feed.
    const handleUrlSubmit = (e) => {
        e.preventDefault();
        const newUrl = e.target.elements.urlInput.value;
        setRssUrl(newUrl);
    };

    return (
        <div className="bg-gray-100 min-h-screen p-4 sm:p-8 font-sans antialiased text-gray-900">
            <div className="max-w-4xl mx-auto">
                <header className="mb-8 text-center">
                    <h1 className="text-3xl sm:text-4xl font-extrabold text-blue-800">RSS Feed Reader</h1>
                    <p className="mt-2 text-gray-600">Enter an RSS feed URL to see the latest articles.</p>
                </header>

                <form onSubmit={handleUrlSubmit} className="mb-8 flex flex-col sm:flex-row items-center space-y-4 sm:space-y-0 sm:space-x-4">
                    <input
                        id="urlInput"
                        type="url"
                        placeholder="e.g., https://www.theverge.com/rss/index.xml"
                        defaultValue={rssUrl}
                        className="flex-1 w-full px-4 py-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors"
                    />
                    <button
                        type="submit"
                        className="w-full sm:w-auto px-6 py-3 bg-blue-600 text-white font-semibold rounded-lg shadow-md hover:bg-blue-700 transition-colors duration-300"
                    >
                        Load Feed
                    </button>
                </form>

                {/* Conditional rendering for loading, error, and content. */}
                {isLoading ? (
                    <div className="flex items-center justify-center py-12">
                        <div className="animate-spin rounded-full h-12 w-12 border-4 border-t-4 border-blue-500 border-opacity-50"></div>
                        <p className="ml-4 text-gray-600">Loading feed...</p>
                    </div>
                ) : error ? (
                    <div className="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 rounded-lg shadow-md" role="alert">
                        <p className="font-bold">Error</p>
                        <p>{error}</p>
                    </div>
                ) : (
                    <div className="grid gap-6">
                        {feedItems.length > 0 ? (
                            feedItems.map((item, index) => (
                                <div key={index} className="bg-white p-6 rounded-lg shadow-lg hover:shadow-xl transition-shadow duration-300">
                                    <h2 className="text-xl sm:text-2xl font-bold mb-2 leading-tight">
                                        <a href={item.link} target="_blank" rel="noopener noreferrer" className="text-blue-700 hover:text-blue-900 transition-colors">
                                            {item.title}
                                        </a>
                                    </h2>
                                    <p className="text-sm text-gray-500 mb-4 italic">
                                        Published: {new Date(item.pubDate).toLocaleString()}
                                    </p>
                                    <p className="text-gray-700 mb-4">{item.description.replace(/(<([^>]+)>)/ig, '')}</p>
                                    <a
                                        href={item.link}
                                        target="_blank"
                                        rel="noopener noreferrer"
                                        className="inline-block px-4 py-2 text-sm font-semibold text-white bg-blue-600 rounded-full hover:bg-blue-700 transition-colors"
                                    >
                                        Read more
                                    </a>
                                </div>
                            ))
                        ) : (
                            <div className="bg-yellow-100 border-l-4 border-yellow-500 text-yellow-700 p-4 rounded-lg shadow-md" role="alert">
                                <p className="font-bold">No Items</p>
                                <p>No items found in this RSS feed.</p>
                            </div>
                        )}
                    </div>
                )}
            </div>
        </div>
    );
};

export default App;
