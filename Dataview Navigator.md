```dataviewjs
// Dataview Navigator: Navigator for notes using tags & web links

// Light Mode Toggle (1 for Light Mode, 0 for Dark Mode)
const LightMode = 0; // Set to 1 for Light Mode

// Query notes from a specified folder
const notes = dv.pages('"Library/Database"') // Write folder directory
    .filter(n => n.file.name && n.link && n.description); // Required fields

// Create the container for the search bar and content
const container = dv.container;

// Base styles for light and dark mode
const baseStyles = {
    backgroundColor: LightMode ? "#ffffff" : "#222222",
    color: LightMode ? "#000000" : "#f5f5f5",
    borderColor: LightMode ? "#ddd" : "#555",
};

// Create search bar container
const searchContainer = document.createElement('div');
searchContainer.style.marginBottom = "10px";

// Main search bar
const searchBar = document.createElement("input");
searchBar.type = "text";
searchBar.placeholder = "Search resource...";
searchBar.style.padding = "10px";
searchBar.style.width = "100%";
searchBar.style.height = "40px";
searchBar.style.border = `1px solid ${baseStyles.borderColor}`;
searchBar.style.borderRadius = "8px";
searchBar.style.backgroundColor = "transparent";
searchBar.style.color = baseStyles.color;
searchContainer.appendChild(searchBar);

// Combined filter container
const filterContainer = document.createElement('div');
filterContainer.style.display = 'flex';
filterContainer.style.alignItems = 'center';
filterContainer.style.marginTop = '10px';

// Filter dropdown
const filterDropdown = document.createElement('select');
filterDropdown.style.padding = "6px";
filterDropdown.style.borderRadius = "8px";
filterDropdown.style.border = `1px solid ${baseStyles.borderColor}`;
filterDropdown.style.marginRight = '10px';
filterDropdown.style.width = '30%';
filterDropdown.style.color = baseStyles.color;
filterDropdown.style.backgroundColor = LightMode ? "#f5f5f5" : "#222222"; 

// Add filter options
const tagOption = document.createElement('option');
tagOption.value = 'tag';
tagOption.textContent = 'Filter by Tag';
filterDropdown.appendChild(tagOption);

const domainOption = document.createElement('option');
domainOption.value = 'domain';
domainOption.textContent = 'Filter by Domain';
filterDropdown.appendChild(domainOption);

// Filter input
const filterInput = document.createElement("input");
filterInput.type = "text";
filterInput.placeholder = "Filter by...";
filterInput.style.padding = "6px"; 
filterInput.style.width = "67%";
filterInput.style.height = "36px"; 
filterInput.style.border = `1px solid ${baseStyles.borderColor}`;
filterInput.style.borderRadius = "8px";
filterInput.style.backgroundColor = "transparent";
filterInput.style.color = baseStyles.color;

filterContainer.appendChild(filterDropdown);
filterContainer.appendChild(filterInput);
searchContainer.appendChild(filterContainer);
container.appendChild(searchContainer);

// Pagination container
let paginationContainer = document.createElement('div');
paginationContainer.className = 'pagination';
paginationContainer.style.display = "flex";
paginationContainer.style.justifyContent = "space-between";
paginationContainer.style.alignItems = "center";
paginationContainer.style.marginTop = "10px"; 
paginationContainer.style.marginBottom = "20px"; 
container.appendChild(paginationContainer); 

// Pagination variables
let currentPage = 1;
const itemsPerPage = 6;
let totalPages = 0;

// Function to render the feed based on filtered data
const renderFeed = (filter = "", filterType = "") => {
    const existingFeed = container.querySelector('.feed-container');
    if (existingFeed) {
        existingFeed.remove();
    }

    const filteredNotes = notes.filter(n => {
        const matchesSearch = n.file.name.toLowerCase().includes(filter) ||
                              (n.category && n.category.toLowerCase().includes(filter)) ||
                              (n.status && n.status.toLowerCase().includes(filter)) ||
                              (n.due && n.due.includes(filter)) ||
                              (n.priority && n.priority.toLowerCase().includes(filter)) ||
                              n.description.toLowerCase().includes(filter);

        const searchText = filterInput.value.toLowerCase();
        const matchesFilterType = (
            filterType === 'tag' && n.tags && n.tags.some(tag => tag.toLowerCase().includes(searchText)) ||
            (filterType === 'domain' && n.link && n.link.includes(searchText))
        );

        return matchesSearch && (!searchText || matchesFilterType);
    });

    totalPages = Math.ceil(filteredNotes.length / itemsPerPage);
    const paginatedNotes = filteredNotes.slice((currentPage - 1) * itemsPerPage, currentPage * itemsPerPage);

    const feedContainer = document.createElement('div');
    feedContainer.className = 'feed-container';
    feedContainer.style.display = "flex";
    feedContainer.style.flexDirection = "column";
    feedContainer.style.gap = "15px";

    if (paginatedNotes.length === 0) {
        const noResultsMessage = document.createElement('div');
        noResultsMessage.textContent = "No results :(";
        noResultsMessage.style.color = baseStyles.color;
        noResultsMessage.style.fontSize = "16px";
        noResultsMessage.style.textAlign = "center"; 
        feedContainer.appendChild(noResultsMessage);
    } else {
        paginatedNotes.forEach(n => {
            const row = document.createElement('div');
            row.style.display = "flex";
            row.style.alignItems = "flex-start"; 

            const isYouTubeVideo = n.link.includes("youtube.com") || n.link.includes("youtu.be");
            const isYouTubePlaylist = n.link.includes("list="); 
            row.style.padding = isYouTubeVideo ? "15px" : "8px"; 
            row.style.borderRadius = "8px";
            row.style.backgroundColor = n.color ? n.color : (LightMode ? "#f9f9f9" : "transparent");
            row.style.boxShadow = "0 4px 8px rgba(0, 0, 0, 0.1)";
            row.style.transition = "background-color 0.3s ease";
            row.style.color = baseStyles.color;

            // Function to adjust color brightness
            function lightenColor(color, percent) {
                const colorElement = document.createElement('div');
                colorElement.style.backgroundColor = color;
                document.body.appendChild(colorElement);
                
                const rgb = window.getComputedStyle(colorElement).backgroundColor.match(/\d+/g);
                document.body.removeChild(colorElement);
                
                if (rgb) {
                    const r = Math.min(255, Math.floor(+rgb[0] + (percent * 255)));
                    const g = Math.min(255, Math.floor(+rgb[1] + (percent * 255)));
                    const b = Math.min(255, Math.floor(+rgb[2] + (percent * 255)));
                    
                    return `rgb(${r}, ${g}, ${b})`;
                }
                return color; 
            }

            // Hover effects
            row.addEventListener("mouseenter", () => {
                const brighterColor = n.color ? lightenColor(n.color, 0.15) : "#ffff";
                row.style.backgroundColor = brighterColor;
            });

            row.addEventListener("mouseleave", () => {
                row.style.backgroundColor = n.color ? n.color : (LightMode ? "#f9f9f9" : "transparent"); 
            });

            // YouTube video thumbnail
            if (isYouTubeVideo) {
                const videoThumbnail = document.createElement('img');
                let videoId = isYouTubePlaylist ? new URL(n.link).searchParams.get("list") : (n.link.includes('v=') ? n.link.split('v=')[1].split('&')[0] : n.link.split('/').pop());
                videoThumbnail.src = `https://img.youtube.com/vi/${videoId}/hqdefault.jpg`;
                videoThumbnail.style.width = "180px";
                videoThumbnail.style.height = "101px";
                videoThumbnail.style.borderRadius = "8px";
                videoThumbnail.style.marginRight = "15px";
                videoThumbnail.style.cursor = "pointer";
                videoThumbnail.style.objectFit = "cover";
                videoThumbnail.addEventListener("click", () => {
                    window.open(n.link, '_blank'); // Open video in a new tab
                });
                row.appendChild(videoThumbnail);
            } else {
                // Show favicon for generic websites
                const favicon = document.createElement('img');
                const faviconSize = "24px"; 
                favicon.src = `https://www.google.com/s2/favicons?domain=${new URL(n.link).hostname}`;
                favicon.style.width = faviconSize;
                favicon.style.height = faviconSize;
                favicon.style.marginRight = "10px";  
                favicon.style.marginTop = "2px"; 
                favicon.style.objectFit = "contain";
                row.appendChild(favicon);
            }

            // Text section for title and description
            const textContent = document.createElement('div');
            textContent.style.flex = "1";
            textContent.style.display = "flex";
            textContent.style.flexDirection = "column";
            textContent.style.gap = "3px"; 

            const title = document.createElement('a');
            title.textContent = n.file.name;
            title.href = n.link; 
            title.style.marginBottom = "2px";
            title.style.fontSize = "16px";
            title.style.color = baseStyles.color; 
            title.style.fontWeight = "bold"; 
            title.style.textDecoration = "none"; 
            title.addEventListener('mouseenter', () => title.style.textDecoration = "underline");
            title.addEventListener('mouseleave', () => title.style.textDecoration = "none");
            textContent.appendChild(title);

            const description = document.createElement('p');
            description.textContent = n.description;
            description.style.fontSize = "14px";
            description.style.color = baseStyles.color; 
            description.style.marginBottom = "10px";
            description.style.lineHeight = "1.4";
            textContent.appendChild(description);

            // Display tags if present
            if (n.tags) {
                const tagsContainer = document.createElement('div');
                tagsContainer.style.display = "flex";
                tagsContainer.style.gap = "8px";
                tagsContainer.style.flexWrap = "wrap";

                n.tags.forEach(tag => {
                    const tagElement = document.createElement('span');
                    tagElement.textContent = `#${tag}`;
                    tagElement.style.padding = "4px 8px";
                    tagElement.style.fontSize = "12px";
                    tagElement.style.borderRadius = "4px";
                    tagElement.style.backgroundColor = "rgba(0, 0, 0, 0.1)"; 
                    tagElement.style.color = LightMode ? "#000000" : "#f5f5f5"; 
                    tagsContainer.appendChild(tagElement);
                });

                textContent.appendChild(tagsContainer);
            }

            row.appendChild(textContent);
            feedContainer.appendChild(row);
        });
    }

    container.appendChild(feedContainer);
    setupPagination(); // Setup pagination after feed rendering
};

// Set up pagination controls
const setupPagination = () => {
    paginationContainer.innerHTML = '';

    const createButton = (text, disabled, onClick) => {
        const button = document.createElement('button');
        button.textContent = text;
        button.disabled = disabled;
        button.style.backgroundColor = LightMode ? "rgba(0, 0, 0, 0.05)" : "rgba(0, 0, 0, 0.2)"; 
        button.style.color = baseStyles.color; 
        button.style.border = "none"; 
        button.style.borderRadius = "12px"; 
        button.style.padding = "8px 14px"; 
        button.style.cursor = "pointer"; 
        button.style.boxShadow = "0 2px 4px rgba(0, 0, 0, 0.2)"; 
        button.style.transition = "background-color 0.2s ease, transform 0.1s"; 

        button.addEventListener('mouseenter', () => {
            button.style.backgroundColor = LightMode ? "rgba(0, 0, 0, 0.2)" : "rgba(0, 0, 0, 0.3)"; 
            button.style.transform = "scale(1.05)"; 
        });
        button.addEventListener('mouseleave', () => {
            button.style.backgroundColor = LightMode ? "rgba(0, 0, 0, 0.1)" : "rgba(0, 0, 0, 0.2)"; 
            button.style.transform = "scale(1)"; 
        });
        button.onclick = onClick; 
        return button;
    };

    // Create pagination buttons
    const firstPageBtn = createButton('First', currentPage === 1, () => {
        currentPage = 1;
        renderFeed(searchBar.value.toLowerCase(), filterDropdown.value);
    });
    paginationContainer.appendChild(firstPageBtn);

    const prevPageBtn = createButton('Prev', currentPage === 1, () => {
        if (currentPage > 1) {
            currentPage--;
            renderFeed(searchBar.value.toLowerCase(), filterDropdown.value);
        }
    });
    paginationContainer.appendChild(prevPageBtn);

    const pageInfo = document.createElement('span');
    pageInfo.textContent = `Page ${currentPage} of ${totalPages}`;
    paginationContainer.appendChild(pageInfo);

    const nextPageBtn = createButton('Next', currentPage === totalPages, () => {
        if (currentPage < totalPages) {
            currentPage++;
            renderFeed(searchBar.value.toLowerCase(), filterDropdown.value);
        }
    });
    paginationContainer.appendChild(nextPageBtn);

    const lastPageBtn = createButton('Last', currentPage === totalPages, () => {
        currentPage = totalPages;
        renderFeed(searchBar.value.toLowerCase(), filterDropdown.value);
    });
    paginationContainer.appendChild(lastPageBtn);
};

// Initial rendering of the feed
renderFeed();

// Debounce function for search input
const debounce = (func, delay) => {
    let timer;
    return function (...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
            func(...args);
        }, delay);
    };
};

// Handle input changes in search and filter
const handleFilterChange = () => {
    const searchText = searchBar.value.toLowerCase();
    const filterType = filterDropdown.value;
    currentPage = 1; 
    renderFeed(searchText, filterType); 
};

// Add event listeners for input changes
searchBar.addEventListener("input", debounce(handleFilterChange, 300));
filterInput.addEventListener("input", debounce(handleFilterChange, 300));
filterDropdown.addEventListener("change", handleFilterChange);
````