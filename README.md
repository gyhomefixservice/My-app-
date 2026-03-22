import React, { useState, useEffect, useMemo } from 'react';
import { 
  Wrench, 
  Droplets, 
  Zap, 
  Layout, 
  DoorOpen, 
  Hammer, 
  PaintBucket, 
  Settings, 
  Phone, 
  MessageCircle, 
  Instagram, 
  Mail, 
  MapPin, 
  Calendar, 
  User, 
  ChevronRight,
  CheckCircle, 
  Clock, 
  AlertCircle,
  Menu,
  X,
  LogIn,
  ClipboardList,
  HardHat,
  Star,
  ShieldCheck,
  TrendingUp
} from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { 
  getFirestore, 
  collection, 
  addDoc, 
  onSnapshot, 
  updateDoc, 
  doc, 
  query, 
  serverTimestamp 
} from 'firebase/firestore';
import { 
  getAuth, 
  signInAnonymously, 
  signInWithCustomToken, 
  onAuthStateChanged 
} from 'firebase/auth';

// --- Firebase Configuration ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'gy-homefix-service';

// --- Constants ---
const SERVICES_LIST = [
  { id: 'ac', name: 'AC Repair', icon: <Zap className="w-6 h-6" />, desc: 'Installation, gas refilling, and deep cleaning.' },
  { id: 'plumbing', name: 'Plumbing', icon: <Droplets className="w-6 h-6" />, desc: 'Leakage repair, pipe fitting, and tap installation.' },
  { id: 'electrical', name: 'Electrical Work', icon: <Zap className="w-6 h-6" />, desc: 'Wiring, switchboards, and appliance repairs.' },
  { id: 'windows', name: 'UPVC & Aluminium Work', icon: <Layout className="w-6 h-6" />, desc: 'Window repairs, sliding tracks, and mesh.' },
  { id: 'doors', name: 'Door Repair', icon: <DoorOpen className="w-6 h-6" />, desc: 'Handle repair, hinges, and alignment.' },
  { id: 'furniture', name: 'Furniture Repair', icon: <Hammer className="w-6 h-6" />, desc: 'Table, chair, and wardrobe maintenance.' },
  { id: 'painting', name: 'Painting', icon: <PaintBucket className="w-6 h-6" />, desc: 'Interior/Exterior painting and wall touch-ups.' },
  { id: 'carpenter', name: 'Carpenter Services', icon: <Wrench className="w-6 h-6" />, desc: 'Custom wood work and home structural repairs.' },
  { id: 'general', name: 'General Maintenance', icon: <Settings className="w-6 h-6" />, desc: 'All types of odd jobs and home care.' },
];

const MECHANICS = [
  { id: 'm1', name: 'Rakesh Kumar' },
  { id: 'm2', name: 'Suresh Patil' },
  { id: 'm3', name: 'Vijay Shinde' },
  { id: 'm4', name: 'Anil Deshmukh' },
];

// --- Components ---

const Navbar = ({ currentView, setView, userRole, logout }) => {
  const [isOpen, setIsOpen] = useState(false);

  const navLinks = [
    { label: 'Home', view: 'home' },
    { label: 'Services', view: 'services' },
    { label: 'About', view: 'about' },
    { label: 'Contact', view: 'contact' },
  ];

  return (
    <nav className="bg-white/80 backdrop-blur-md shadow-sm sticky top-0 z-50 border-b border-slate-100">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between h-20">
          <div className="flex items-center cursor-pointer" onClick={() => setView('home')}>
            <div className="flex items-center space-x-3">
              <div className="bg-gradient-to-br from-blue-600 to-blue-800 p-2 rounded-xl shadow-lg shadow-blue-500/20">
                <Wrench className="text-white w-6 h-6" />
              </div>
              <div className="flex flex-col">
                <span className="text-xl font-black tracking-tighter leading-none text-slate-900">
                  GY <span className="text-orange-500">HOMEFIX</span>
                </span>
                <span className="text-[10px] uppercase tracking-[0.2em] font-bold text-slate-400">Service • Thane</span>
              </div>
            </div>
          </div>

          {/* Desktop Nav */}
          <div className="hidden md:flex items-center space-x-8">
            {navLinks.map((link) => (
              <button
                key={link.view}
                onClick={() => setView(link.view)}
                className={`text-sm font-bold tracking-wide transition-all ${
                  currentView === link.view ? 'text-blue-600 border-b-2 border-blue-600 pb-1' : 'text-slate-500 hover:text-orange-500'
                }`}
              >
                {link.label}
              </button>
            ))}
            
            {userRole ? (
              <button 
                onClick={() => setView(userRole === 'admin' ? 'admin' : 'mechanic')}
                className="bg-slate-900 text-white px-5 py-2.5 rounded-xl text-sm font-bold hover:bg-slate-800 flex items-center gap-2 shadow-lg shadow-slate-900/10 transition-transform active:scale-95"
              >
                {userRole === 'admin' ? <Settings size={16}/> : <HardHat size={16}/>}
                Panel
              </button>
            ) : (
              <button 
                onClick={() => setView('login')}
                className="bg-blue-600 hover:bg-blue-700 text-white px-6 py-2.5 rounded-xl text-sm font-bold shadow-lg shadow-blue-600/20 transition-all flex items-center gap-2"
              >
                <LogIn size={18} /> Staff
              </button>
            )}
          </div>

          {/* Mobile menu button */}
          <div className="md:hidden flex items-center">
            <button onClick={() => setIsOpen(!isOpen)} className="text-slate-600 p-2 hover:bg-slate-100 rounded-lg">
              {isOpen ? <X size={24} /> : <Menu size={24} />}
            </button>
          </div>
        </div>
      </div>

      {/* Mobile Nav */}
      {isOpen && (
        <div className="md:hidden bg-white border-t border-slate-100 p-4 space-y-2 animate-in slide-in-from-top">
          {navLinks.map((link) => (
            <button
              key={link.view}
              onClick={() => { setView(link.view); setIsOpen(false); }}
              className={`block w-full text-left px-4 py-3 rounded-xl text-base font-bold transition-colors ${
                currentView === link.view ? 'bg-blue-50 text-blue-600' : 'text-slate-600 hover:bg-slate-50'
              }`}
            >
              {link.label}
            </button>
          ))}
          <div className="h-px bg-slate-100 my-2"></div>
          <button
             onClick={() => { setView('login'); setIsOpen(false); }}
             className="block w-full text-left px-4 py-3 text-base font-bold text-blue-600 hover:bg-blue-50 rounded-xl"
          >
            Staff Access
          </button>
        </div>
      )}
    </nav>
  );
};

const Hero = ({ setView }) => (
  <div className="relative bg-slate-950 text-white pt-20 pb-32 lg:pt-32 lg:pb-48 overflow-hidden">
    <div className="absolute inset-0 z-0">
      <div className="absolute top-0 right-0 w-[800px] h-[800px] bg-blue-600/10 rounded-full blur-[120px] -translate-y-1/2 translate-x-1/2"></div>
      <div className="absolute bottom-0 left-0 w-[600px] h-[600px] bg-orange-600/10 rounded-full blur-[100px] translate-y-1/2 -translate-x-1/2"></div>
    </div>
    <div className="max-w-7xl mx-auto px-4 relative z-10 text-center lg:text-left flex flex-col lg:flex-row items-center gap-16">
      <div className="lg:w-1/2">
        <div className="inline-flex items-center gap-2 py-1.5 px-3 bg-white/5 border border-white/10 rounded-full mb-8">
          <span className="flex h-2 w-2 rounded-full bg-blue-500 animate-pulse"></span>
          <span className="text-xs font-bold text-slate-300 uppercase tracking-widest">Available 24/7 in Thane</span>
        </div>
        <h1 className="text-5xl lg:text-7xl font-black mb-8 leading-[1.1] tracking-tight">
          Reliable <span className="text-orange-500">HomeFix</span> Services.
        </h1>
        <p className="text-xl text-slate-400 mb-10 leading-relaxed max-w-2xl mx-auto lg:mx-0">
          Professional plumbing, electrical, and maintenance services delivered right to your doorstep with a <span className="text-white font-bold">100% satisfaction guarantee.</span>
        </p>
        <div className="flex flex-col sm:flex-row gap-5 justify-center lg:justify-start">
          <button 
            onClick={() => setView('book')}
            className="group px-10 py-5 bg-orange-500 hover:bg-orange-600 text-white rounded-2xl font-black text-lg shadow-2xl shadow-orange-500/40 transition-all hover:-translate-y-1 flex items-center justify-center gap-2"
          >
            Book Appointment <ChevronRight size={20} className="group-hover:translate-x-1 transition-transform"/>
          </button>
          <button 
            onClick={() => setView('services')}
            className="px-10 py-5 bg-white/5 hover:bg-white/10 text-white border border-white/10 rounded-2xl font-black text-lg backdrop-blur-md transition-all"
          >
            Explore Services
          </button>
        </div>
        
        <div className="mt-12 flex flex-wrap justify-center lg:justify-start gap-8 opacity-60">
          {[
            { icon: <ShieldCheck size={18}/>, text: 'Verified Experts' },
            { icon: <Star size={18}/>, text: '4.9 Rating' },
            { icon: <Clock size={18}/>, text: '60-min Arrival' },
          ].map((item, i) => (
            <div key={i} className="flex items-center gap-2 text-sm font-semibold">
              {item.icon} {item.text}
            </div>
          ))}
        </div>
      </div>
      
      {/* Decorative Elements */}
      <div className="lg:w-1/2 hidden lg:block relative">
        <div className="relative z-10 grid grid-cols-2 gap-4">
          <div className="space-y-4 pt-12">
            <div className="h-64 bg-gradient-to-br from-blue-600/20 to-blue-800/20 border border-white/10 rounded-[40px] flex items-center justify-center backdrop-blur-sm">
               <Wrench className="text-blue-500 w-24 h-24" />
            </div>
            <div className="h-48 bg-gradient-to-br from-orange-500/20 to-orange-700/20 border border-white/10 rounded-[40px] flex items-center justify-center backdrop-blur-sm">
               <Zap className="text-orange-500 w-16 h-16" />
            </div>
          </div>
          <div className="space-y-4">
            <div className="h-48 bg-slate-900 border border-white/10 rounded-[40px] p-8">
               <div className="flex gap-1 mb-4">
                 {[1,2,3,4,5].map(s => <Star key={s} size={16} fill="#f97316" className="text-orange-500" />)}
               </div>
               <p className="text-sm font-medium italic text-slate-400">"The fastest and most reliable plumbing service I've used in Thane. Highly recommended!"</p>
               <p className="text-xs font-bold mt-4 text-white">— Priya Sharma</p>
            </div>
            <div className="h-64 bg-gradient-to-br from-slate-800 to-slate-900 border border-white/10 rounded-[40px] flex items-center justify-center">
               <Hammer className="text-slate-600 w-20 h-20" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
);

const ServiceCard = ({ service, setView }) => (
  <div 
    onClick={() => setView('book')}
    className="bg-white p-8 rounded-[32px] shadow-sm border border-slate-100 hover:shadow-2xl hover:shadow-blue-500/10 hover:border-blue-200 transition-all group cursor-pointer relative overflow-hidden"
  >
    <div className="absolute top-0 right-0 w-24 h-24 bg-slate-50 rounded-bl-[60px] -z-0 group-hover:bg-blue-50 transition-colors"></div>
    <div className="relative z-10">
      <div className="w-16 h-16 bg-slate-50 text-slate-900 rounded-2xl flex items-center justify-center mb-6 group-hover:bg-blue-600 group-hover:text-white transition-all transform group-hover:rotate-6">
        {service.icon}
      </div>
      <h3 className="text-2xl font-black text-slate-900 mb-3 tracking-tight">{service.name}</h3>
      <p className="text-slate-500 text-sm leading-relaxed mb-6">{service.desc}</p>
      <div className="flex items-center text-blue-600 text-xs font-black uppercase tracking-widest group-hover:gap-2 transition-all">
        Book Now <ChevronRight size={16} />
      </div>
    </div>
  </div>
);

const BookingForm = ({ setNotification, initialService = '' }) => {
  const [formData, setFormData] = useState({
    name: '',
    phone: '',
    address: '',
    service: initialService,
    date: '',
    message: ''
  });
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    try {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'bookings'), {
        ...formData,
        status: 'Pending',
        createdAt: serverTimestamp(),
        assignedMechanic: null,
      });
      setNotification({ type: 'success', message: 'Booking confirmed! We will contact you soon.' });
      setFormData({ name: '', phone: '', address: '', service: '', date: '', message: '' });
    } catch (error) {
      setNotification({ type: 'error', message: 'Booking failed. Check connection.' });
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto bg-white rounded-[40px] shadow-2xl overflow-hidden border border-slate-100 flex flex-col md:flex-row">
      <div className="md:w-1/3 bg-slate-900 p-10 text-white flex flex-col justify-between">
        <div>
          <h2 className="text-3xl font-black mb-4">Book a Repair</h2>
          <p className="text-slate-400 text-sm leading-relaxed mb-8">Tell us what needs fixing, and our verified professional will visit your home at your scheduled time.</p>
          <ul className="space-y-4">
            {[
              { icon: <CheckCircle size={18} className="text-green-500"/>, text: 'No Advance Fees' },
              { icon: <CheckCircle size={18} className="text-green-500"/>, text: 'Verified Experts Only' },
              { icon: <CheckCircle size={18} className="text-green-500"/>, text: 'Warranty on Spares' },
            ].map((item, i) => (
              <li key={i} className="flex items-center gap-3 text-sm font-bold text-slate-300">
                {item.icon} {item.text}
              </li>
            ))}
          </ul>
        </div>
        <div className="mt-12 bg-white/5 p-4 rounded-2xl border border-white/10">
           <p className="text-[10px] uppercase tracking-widest font-bold text-slate-500 mb-2">Emergency Support</p>
           <p className="text-lg font-bold">+91 9984281797</p>
        </div>
      </div>
      
      <form onSubmit={handleSubmit} className="md:w-2/3 p-10 space-y-6">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div className="space-y-2">
            <label className="text-xs font-black uppercase tracking-widest text-slate-500 ml-1">Full Name</label>
            <div className="relative">
              <User className="absolute left-4 top-1/2 -translate-y-1/2 text-slate-300" size={20} />
              <input 
                required
                type="text"
                placeholder="Ex: John Doe"
                className="w-full pl-12 pr-4 py-4 bg-slate-50 border border-slate-100 rounded-2xl focus:ring-4 focus:ring-blue-500/10 focus:border-blue-500 outline-none transition-all font-medium"
                value={formData.name}
                onChange={e => setFormData({...formData, name: e.target.value})}
              />
            </div>
          </div>
          <div className="space-y-2">
            <label className="text-xs font-black uppercase tracking-widest text-slate-500 ml-1">Phone Number</label>
            <div className="relative">
              <Phone className="absolute left-4 top-1/2 -translate-y-1/2 text-slate-300" size={20} />
              <input 
                required
                type="tel"
                placeholder="Ex: 9984281797"
                className="w-full pl-12 pr-4 py-4 bg-slate-50 border border-slate-100 rounded-2xl focus:ring-4 focus:ring-blue-500/10 focus:border-blue-500 outline-none transition-all font-medium"
                value={formData.phone}
                onChange={e => setFormData({...formData, phone: e.target.value})}
              />
            </div>
          </div>
        </div>

        <div className="space-y-2">
          <label className="text-xs font-black uppercase tracking-widest text-slate-500 ml-1">Home Address (Thane)</label>
          <div className="relative">
            <MapPin className="absolute left-4 top-1/2 -translate-y-1/2 text-slate-300" size={20} />
            <input 
              required
              type="text"
              placeholder="Building, Flat No, Landmark"
              className="w-full pl-12 pr-4 py-4 bg-slate-50 border border-slate-100 rounded-2xl focus:ring-4 focus:ring-blue-500/10 focus:border-blue-500 outline-none transition-all font-medium"
              value={formData.address}
              onChange={e => setFormData({...formData, address: e.target.value})}
            />
          </div>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div className="space-y-2">
            <label className="text-xs font-black uppercase tracking-widest text-slate-500 ml-1">Service Type</label>
            <select 
              required
              className="w-full px-4 py-4 bg-slate-50 border border-slate-100 rounded-2xl focus:ring-4 focus:ring-blue-500/10 focus:border-blue-500 outline-none transition-all font-bold appearance-none cursor-pointer"
              value={formData.service}
              onChange={e => setFormData({...formData, service: e.target.value})}
            >
              <option value="">Select Service</option>
              {SERVICES_LIST.map(s => <option key={s.id} value={s.name}>{s.name}</option>)}
            </select>
          </div>
          <div className="space-y-2">
            <label className="text-xs font-black uppercase tracking-widest text-slate-500 ml-1">Preferred Date</label>
            <div className="relative">
              <Calendar className="absolute left-4 top-1/2 -translate-y-1/2 text-slate-300" size={20} />
              <input 
                required
                type="date"
                className="w-full pl-12 pr-4 py-4 bg-slate-50 border border-slate-100 rounded-2xl focus:ring-4 focus:ring-blue-500/10 focus:border-blue-500 outline-none transition-all font-bold"
                value={formData.date}
                onChange={e => setFormData({...formData, date: e.target.value})}
              />
            </div>
          </div>
        </div>

        <button 
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 hover:bg-blue-700 text-white font-black py-5 rounded-2xl transition-all shadow-xl shadow-blue-600/30 flex items-center justify-center gap-2 transform active:scale-[0.98]"
        >
          {loading ? 'Confirming...' : 'Schedule Service Now'}
        </button>
      </form>
    </div>
  );
};

const AdminDashboard = ({ bookings, updateStatus, assignMechanic }) => {
  const stats = useMemo(() => ({
    total: bookings.length,
    pending: bookings.filter(b => b.status === 'Pending').length,
    active: bookings.filter(b => b.status === 'Accepted').length,
    done: bookings.filter(b => b.status === 'Completed').length,
  }), [bookings]);

  return (
    <div className="bg-slate-50 min-h-screen p-4 md:p-10">
      <div className="max-w-7xl mx-auto">
        <div className="flex flex-col md:flex-row justify-between items-start md:items-center mb-10 gap-4">
          <div>
            <h1 className="text-3xl font-black text-slate-900 tracking-tight">Admin Portal</h1>
            <p className="text-slate-500 font-medium">Managing home service requests across Thane.</p>
          </div>
          <div className="flex gap-2">
            <div className="bg-white p-3 px-6 rounded-2xl shadow-sm border border-slate-100 text-center">
              <p className="text-[10px] uppercase font-bold text-slate-400">Total Bookings</p>
              <p className="text-xl font-black text-blue-600">{stats.total}</p>
            </div>
          </div>
        </div>

        <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-10">
          {[
            { label: 'Pending', count: stats.pending, color: 'text-orange-500', bg: 'bg-orange-50' },
            { label: 'In Progress', count: stats.active, color: 'text-blue-500', bg: 'bg-blue-50' },
            { label: 'Completed', count: stats.done, color: 'text-green-500', bg: 'bg-green-50' },
            { label: 'Growth', count: '+12%', color: 'text-purple-500', bg: 'bg-purple-50' },
          ].map(stat => (
            <div key={stat.label} className={`${stat.bg} p-6 rounded-3xl border border-white flex flex-col items-center justify-center`}>
              <p className={`text-2xl font-black ${stat.color}`}>{stat.count}</p>
              <p className="text-[10px] font-black uppercase tracking-widest text-slate-400 mt-1">{stat.label}</p>
            </div>
          ))}
        </div>

        <div className="bg-white rounded-[32px] shadow-sm border border-slate-100 overflow-hidden">
          <div className="overflow-x-auto">
            <table className="w-full text-left">
              <thead className="bg-slate-50/50 border-b border-slate-100">
                <tr>
                  <th className="px-8 py-5 text-xs font-black text-slate-400 uppercase tracking-widest">Customer</th>
                  <th className="px-8 py-5 text-xs font-black text-slate-400 uppercase tracking-widest">Service & Date</th>
                  <th className="px-8 py-5 text-xs font-black text-slate-400 uppercase tracking-widest">Assignment</th>
                  <th className="px-8 py-5 text-xs font-black text-slate-400 uppercase tracking-widest">Status</th>
                  <th className="px-8 py-5 text-xs font-black text-slate-400 uppercase tracking-widest">Manage</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-50">
                {bookings.map((booking) => (
                  <tr key={booking.id} className="hover:bg-slate-50/50 transition-colors">
                    <td className="px-8 py-6">
                      <p className="font-bold text-slate-900">{booking.name}</p>
                      <p className="text-xs font-medium text-slate-500 mt-1">{booking.phone}</p>
                      <div className="flex items-center gap-1 text-[10px] text-slate-400 mt-1">
                        <MapPin size={10}/> <span className="truncate max-w-[120px]">{booking.address}</span>
                      </div>
                    </td>
                    <td className="px-8 py-6">
                      <span className="inline-block px-3 py-1 bg-slate-100 text-slate-700 text-[10px] font-black uppercase rounded-lg mb-1">{booking.service}</span>
                      <p className="text-sm font-bold text-slate-600">{booking.date}</p>
                    </td>
                    <td className="px-8 py-6">
                      <select 
                        className="text-xs font-bold bg-white border border-slate-200 rounded-xl p-2 outline-none focus:ring-2 focus:ring-blue-500/20"
                        value={booking.assignedMechanic || ''}
                        onChange={(e) => assignMechanic(booking.id, e.target.value)}
                      >
                        <option value="">Select Mechanic</option>
                        {MECHANICS.map(m => <option key={m.id} value={m.name}>{m.name}</option>)}
                      </select>
                    </td>
                    <td className="px-8 py-6">
                      <span className={`px-3 py-1.5 rounded-full text-[10px] font-black uppercase tracking-widest ${
                        booking.status === 'Completed' ? 'bg-green-100 text-green-700' : 
                        booking.status === 'Accepted' ? 'bg-blue-100 text-blue-700' : 'bg-orange-100 text-orange-700'
                      }`}>
                        {booking.status}
                      </span>
                    </td>
                    <td className="px-8 py-6">
                      <div className="flex gap-2">
                        <button 
                          onClick={() => updateStatus(booking.id, 'Accepted')}
                          className="p-2 hover:bg-blue-50 text-blue-600 rounded-xl transition-colors"
                          title="Approve"
                        >
                          <CheckCircle size={20} />
                        </button>
                        <button 
                          onClick={() => updateStatus(booking.id, 'Completed')}
                          className="p-2 hover:bg-green-50 text-green-600 rounded-xl transition-colors"
                          title="Mark Complete"
                        >
                          <CheckCircle size={20} />
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
                {bookings.length === 0 && (
                  <tr>
                    <td colSpan="5" className="px-8 py-20 text-center text-slate-400 font-bold italic">No active bookings found.</td>
                  </tr>
                )}
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  );
};

const MechanicPortal = ({ bookings, updateStatus }) => {
  const [selectedMechanic, setSelectedMechanic] = useState('');
  
  const assignedJobs = useMemo(() => {
    return bookings.filter(b => b.assignedMechanic === selectedMechanic && b.status !== 'Completed');
  }, [bookings, selectedMechanic]);

  if (!selectedMechanic) {
    return (
      <div className="flex flex-col items-center justify-center min-h-[70vh] p-4 bg-slate-50">
        <div className="bg-white p-10 rounded-[40px] shadow-2xl border border-slate-100 w-full max-w-sm text-center">
          <div className="bg-blue-600 w-20 h-20 rounded-3xl flex items-center justify-center mx-auto mb-8 shadow-xl shadow-blue-500/30">
            <HardHat className="text-white" size={40} />
          </div>
          <h2 className="text-3xl font-black mb-2">Staff Login</h2>
          <p className="text-slate-400 text-sm mb-8 font-medium">Select your name to access your jobs.</p>
          <div className="space-y-4">
            <select 
              className="w-full p-4 bg-slate-50 border border-slate-100 rounded-2xl outline-none focus:ring-4 focus:ring-blue-500/10 font-bold appearance-none text-center"
              onChange={(e) => setSelectedMechanic(e.target.value)}
            >
              <option value="">Choose your name</option>
              {MECHANICS.map(m => <option key={m.id} value={m.name}>{m.name}</option>)}
            </select>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="max-w-4xl mx-auto p-4 md:p-10 space-y-8 bg-slate-50 min-h-screen">
      <div className="flex justify-between items-center bg-white p-8 rounded-3xl shadow-sm border border-slate-100">
        <div>
          <h2 className="text-2xl font-black text-slate-900">Hey, {selectedMechanic}!</h2>
          <p className="text-slate-500 font-bold text-sm">{assignedJobs.length} active jobs for you.</p>
        </div>
        <button 
          onClick={() => setSelectedMechanic('')}
          className="bg-slate-50 text-slate-400 p-2 rounded-xl hover:text-red-500 transition-colors"
        >
          <LogIn size={20} className="rotate-180" />
        </button>
      </div>

      <div className="space-y-6">
        {assignedJobs.map(job => (
          <div key={job.id} className="bg-white p-8 rounded-[32px] shadow-sm border border-slate-100 group">
            <div className="flex justify-between items-start mb-6">
              <span className="bg-blue-50 text-blue-600 px-4 py-1.5 rounded-full text-[10px] font-black uppercase tracking-widest">{job.service}</span>
              <div className="text-slate-400 text-xs font-bold flex items-center gap-2 bg-slate-50 px-3 py-1.5 rounded-lg">
                <Clock size={14}/> {job.date}
              </div>
            </div>
            <h3 className="text-2xl font-black text-slate-900 mb-3">{job.name}</h3>
            <div className="flex items-start gap-3 text-slate-500 mb-8">
              <MapPin className="shrink-0 mt-1 text-slate-300" size={18}/>
              <p className="text-sm font-medium leading-relaxed">{job.address}</p>
            </div>
            <div className="grid grid-cols-2 gap-4">
              <a href={`tel:${job.phone}`} className="bg-slate-900 text-white p-4 rounded-2xl flex items-center justify-center gap-2 text-sm font-black shadow-lg shadow-slate-900/10 hover:bg-slate-800 transition-all">
                <Phone size={18}/> Call Customer
              </a>
              <button 
                onClick={() => updateStatus(job.id, 'Completed')}
                className="bg-green-600 text-white p-4 rounded-2xl flex items-center justify-center gap-2 text-sm font-black shadow-lg shadow-green-600/20 hover:bg-green-700 transition-all"
              >
                <CheckCircle size={18}/> Done
              </button>
            </div>
          </div>
        ))}
        {assignedJobs.length === 0 && (
          <div className="text-center py-32 bg-white rounded-[40px] shadow-sm border-2 border-dashed border-slate-100">
            <div className="w-20 h-20 bg-slate-50 rounded-full flex items-center justify-center mx-auto mb-6">
               <ClipboardList className="text-slate-200" size={40} />
            </div>
            <p className="text-slate-400 font-bold italic">No pending jobs for today. Great work!</p>
          </div>
        )}
      </div>
    </div>
  );
};

export default function App() {
  const [view, setView] = useState('home');
  const [user, setUser] = useState(null);
  const [bookings, setBookings] = useState([]);
  const [notification, setNotification] = useState(null);
  const [userRole, setUserRole] = useState(null);

  // Auth Initialization
  useEffect(() => {
    const initAuth = async () => {
      if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
        await signInWithCustomToken(auth, __initial_auth_token);
      } else {
        await signInAnonymously(auth);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, (u) => {
      setUser(u);
    });
    return () => unsubscribe();
  }, []);

  // Data Listening
  useEffect(() => {
    if (!user) return;
    const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'bookings'));
    const unsubscribe = onSnapshot(q, (snapshot) => {
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      // Sort manually as we can't use complex queries
      setBookings(data.sort((a, b) => {
         const dateA = a.createdAt?.toMillis() || 0;
         const dateB = b.createdAt?.toMillis() || 0;
         return dateB - dateA;
      }));
    }, (error) => console.error("Snapshot error:", error));
    return () => unsubscribe();
  }, [user]);

  // Status Handlers
  const updateBookingStatus = async (id, status) => {
    try {
      const bookingRef = doc(db, 'artifacts', appId, 'public', 'data', 'bookings', id);
      await updateDoc(bookingRef, { status });
      setNotification({ type: 'success', message: `Job marked as ${status}` });
    } catch (e) {
      setNotification({ type: 'error', message: 'Failed to update status' });
    }
  };

  const assignMechanic = async (id, mechanicName) => {
    try {
      const bookingRef = doc(db, 'artifacts', appId, 'public', 'data', 'bookings', id);
      await updateDoc(bookingRef, { 
        assignedMechanic: mechanicName, 
        status: mechanicName ? 'Accepted' : 'Pending' 
      });
      setNotification({ type: 'success', message: mechanicName ? `Assigned to ${mechanicName}` : 'Mechanic unassigned' });
    } catch (e) {
      setNotification({ type: 'error', message: 'Assignment failed' });
    }
  };

  // Toast Timer
  useEffect(() => {
    if (notification) {
      const timer = setTimeout(() => setNotification(null), 3500);
      return () => clearTimeout(timer);
    }
  }, [notification]);

  const renderView = () => {
    switch (view) {
      case 'home':
        return (
          <>
            <Hero setView={setView} />
            <section className="py-24 max-w-7xl mx-auto px-4">
              <div className="flex flex-col md:flex-row justify-between items-end mb-16 gap-4">
                <div className="text-left">
                  <span className="text-orange-500 font-black uppercase tracking-[0.3em] text-[10px] mb-4 block">Premium Solutions</span>
                  <h2 className="text-4xl lg:text-5xl font-black text-slate-900 leading-tight">Our Most Trusted <br/>Fixing Services</h2>
                </div>
                <button 
                  onClick={() => setView('services')}
                  className="inline-flex items-center gap-2 text-blue-600 font-black text-xs uppercase tracking-widest hover:translate-x-1 transition-all"
                >
                  View Catalog <ChevronRight size={16}/>
                </button>
              </div>
              <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
                {SERVICES_LIST.slice(0, 6).map(service => (
                  <ServiceCard key={service.id} service={service} setView={setView} />
                ))}
              </div>
              
              {/* Trust Section */}
              <div className="mt-32 p-12 bg-slate-950 rounded-[60px] relative overflow-hidden flex flex-col lg:flex-row items-center gap-12">
                 <div className="absolute top-0 right-0 w-64 h-64 bg-blue-600/20 rounded-full blur-[80px]"></div>
                 <div className="lg:w-2/3 relative z-10 text-center lg:text-left">
                   <h2 className="text-3xl lg:text-4xl font-black text-white mb-6">Experience Professional Quality in Every Corner of Thane.</h2>
                   <p className="text-slate-400 text-lg leading-relaxed mb-10">We don't just fix homes; we build trust through transparency, punctuality, and skilled workmanship.</p>
                   <div className="flex flex-wrap justify-center lg:justify-start gap-12">
                     <div><p className="text-3xl font-black text-white">2.5k+</p><p className="text-xs uppercase tracking-widest font-bold text-slate-500">Jobs Done</p></div>
                     <div><p className="text-3xl font-black text-white">50+</p><p className="text-xs uppercase tracking-widest font-bold text-slate-500">Experts</p></div>
                     <div><p className="text-3xl font-black text-white">4.9/5</p><p className="text-xs uppercase tracking-widest font-bold text-slate-500">Rating</p></div>
                   </div>
                 </div>
                 <div className="lg:w-1/3 text-center">
                    <button 
                      onClick={() => setView('book')}
                      className="px-12 py-6 bg-white text-slate-950 rounded-3xl font-black text-xl hover:scale-105 transition-all shadow-2xl"
                    >
                      Hire Us Now
                    </button>
                 </div>
              </div>
            </section>
          </>
        );
      case 'services':
        return (
          <div className="py-24 max-w-7xl mx-auto px-4">
            <div className="text-center mb-20">
              <h1 className="text-5xl font-black text-slate-900 mb-6 tracking-tight">Full Service Catalog</h1>
              <p className="text-slate-500 max-w-2xl mx-auto font-medium">From minor repairs to major installations, our professionals in Thane handle everything with care.</p>
            </div>
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
              {SERVICES_LIST.map(service => (
                <ServiceCard key={service.id} service={service} setView={setView} />
              ))}
            </div>
          </div>
        );
      case 'book':
        return (
          <div className="py-24 px-4 bg-slate-50 min-h-[80vh]">
            <div className="text-center mb-16">
              <h1 className="text-4xl font-black text-slate-900 mb-4">Quick Appointment</h1>
              <p className="text-slate-500 font-bold">Secure your slot in just 60 seconds.</p>
            </div>
            <BookingForm setNotification={setNotification} />
          </div>
        );
      case 'contact':
        return (
          <div className="py-24 max-w-7xl mx-auto px-4">
            <div className="grid grid-cols-1 md:grid-cols-2 gap-20">
              <div>
                <h2 className="text-5xl font-black mb-8 leading-tight">We're Here <br/><span className="text-blue-600">to Help.</span></h2>
                <p className="text-slate-500 mb-12 text-lg leading-relaxed font-medium">
                  Have an emergency or a unique project? Reach out to our 24/7 support line. We prioritize urgent repairs in Thane Azad Nagar area.
                </p>
                <div className="grid grid-cols-1 sm:grid-cols-2 gap-6">
                  {[
                    { icon: <Phone size={24}/>, label: 'Call Support', value: '9984281797', color: 'text-blue-600', bg: 'bg-blue-50' },
                    { icon: <MessageCircle size={24}/>, label: 'WhatsApp', value: '7619049767', color: 'text-green-600', bg: 'bg-green-50' },
                    { icon: <Mail size={24}/>, label: 'Email Us', value: 'gyhomefixservice@gmail.com', color: 'text-red-500', bg: 'bg-red-50' },
                    { icon: <Instagram size={24}/>, label: 'Follow Us', value: '@gyhomefixservice', color: 'text-pink-500', bg: 'bg-pink-50' },
                  ].map((contact, i) => (
                    <div key={i} className="p-6 bg-white rounded-3xl border border-slate-100 shadow-sm">
                      <div className={`${contact.bg} ${contact.color} w-12 h-12 rounded-2xl flex items-center justify-center mb-4`}>
                        {contact.icon}
                      </div>
                      <p className="text-[10px] font-black uppercase tracking-widest text-slate-400 mb-1">{contact.label}</p>
                      <p className="text-sm font-black text-slate-900">{contact.value}</p>
                    </div>
                  ))}
                </div>
              </div>
              <div className="bg-slate-100 rounded-[60px] p-12 flex flex-col items-center justify-center text-center relative overflow-hidden group">
                 <div className="absolute inset-0 bg-blue-600 opacity-0 group-hover:opacity-5 transition-opacity"></div>
                 <MapPin className="text-blue-600 mb-8 transform group-hover:-translate-y-2 transition-transform" size={80} />
                 <h3 className="text-3xl font-black mb-4">Our Main Office</h3>
                 <p className="text-slate-500 text-lg font-medium leading-relaxed">
                   Bramhand Azad Nagar, Thane,<br/>
                   Maharashtra, India - 400607
                 </p>
                 <div className="mt-12 w-full h-48 bg-slate-200 rounded-[40px] flex items-center justify-center text-slate-400 font-black uppercase tracking-widest text-xs border border-slate-300">
                    Map View
                 </div>
              </div>
            </div>
          </div>
        );
      case 'about':
        return (
          <div className="py-24 max-w-5xl mx-auto px-4">
            <div className="text-center mb-20">
              <span className="bg-blue-50 text-blue-600 px-4 py-1.5 rounded-full text-[10px] font-black uppercase tracking-widest mb-6 inline-block">Our Story</span>
              <h1 className="text-5xl font-black mb-8">Building a Better <br/>Repair Experience.</h1>
              <p className="text-slate-500 text-xl font-medium leading-relaxed max-w-3xl mx-auto">
                GY HomeFix Service was born out of a desire to bring honesty back to home repairs. We vet every technician and use only high-quality materials to ensure your home stays in perfect shape.
              </p>
            </div>
            
            <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
               {[
                 { icon: <ShieldCheck size={40}/>, title: 'Quality First', desc: 'We never compromise on materials or safety.', color: 'text-blue-600' },
                 { icon: <TrendingUp size={40}/>, title: 'Thane First', desc: 'Deeply rooted in the local community of Thane.', color: 'text-orange-500' },
                 { icon: <User size={40}/>, title: 'Staff Vetting', desc: 'Stringent background checks for your security.', color: 'text-green-500' },
               ].map((box, i) => (
                 <div key={i} className="p-10 bg-white rounded-[40px] border border-slate-100 shadow-sm text-center">
                   <div className={`${box.color} mb-6 flex justify-center`}>{box.icon}</div>
                   <h4 className="text-xl font-black mb-3">{box.title}</h4>
                   <p className="text-sm text-slate-400 leading-relaxed font-medium">{box.desc}</p>
                 </div>
               ))}
            </div>
          </div>
        );
      case 'login':
        return (
          <div className="min-h-[75vh] flex items-center justify-center p-4 bg-slate-50">
            <div className="bg-white p-12 rounded-[50px] shadow-2xl border border-slate-100 w-full max-w-md">
              <div className="text-center mb-10">
                <h2 className="text-3xl font-black text-slate-900 tracking-tight">Staff Access</h2>
                <p className="text-slate-400 font-bold mt-2">Internal portal for GY HomeFix team</p>
              </div>
              <div className="space-y-4">
                <button 
                  onClick={() => { setUserRole('admin'); setView('admin'); }}
                  className="w-full py-5 px-6 bg-slate-950 text-white rounded-3xl font-black flex items-center justify-center gap-4 hover:bg-slate-800 transition-all shadow-xl shadow-slate-900/10 active:scale-95"
                >
                  <Settings size={22}/> Admin Console
                </button>
                <div className="relative py-4">
                  <div className="absolute inset-0 flex items-center"><span className="w-full border-t border-slate-100"></span></div>
                  <div className="relative flex justify-center text-[10px] font-black uppercase tracking-[0.4em] text-slate-300"><span className="bg-white px-4">Verified Access</span></div>
                </div>
                <button 
                  onClick={() => { setUserRole('mechanic'); setView('mechanic'); }}
                  className="w-full py-5 px-6 bg-blue-600 text-white rounded-3xl font-black flex items-center justify-center gap-4 hover:bg-blue-700 transition-all shadow-xl shadow-blue-600/20 active:scale-95"
                >
                  <HardHat size={22}/> Mechanic Login
                </button>
              </div>
              <p className="text-center text-[10px] text-slate-300 font-bold mt-10 uppercase tracking-widest">Authorized Personnel Only</p>
            </div>
          </div>
        );
      case 'admin':
        return <AdminDashboard bookings={bookings} updateStatus={updateBookingStatus} assignMechanic={assignMechanic} />;
      case 'mechanic':
        return <MechanicPortal bookings={bookings} updateStatus={updateBookingStatus} />;
      default:
        return <div className="p-20 text-center font-black text-slate-300">Section Under Maintenance</div>;
    }
  };

  return (
    <div className="min-h-screen bg-white font-sans text-slate-950 selection:bg-blue-100 selection:text-blue-900">
      <Navbar currentView={view} setView={setView} userRole={userRole} />
      
      <main className="animate-in fade-in duration-700">
        {renderView()}
      </main>

      <footer className="bg-slate-950 text-white pt-24 pb-12">
        <div className="max-w-7xl mx-auto px-4">
          <div className="grid grid-cols-1 md:grid-cols-4 gap-16 mb-20">
            <div className="col-span-1 md:col-span-1">
              <div className="flex items-center space-x-3 mb-8">
                <div className="bg-blue-600 p-2 rounded-xl">
                  <Wrench className="text-white w-6 h-6" />
                </div>
                <span className="text-2xl font-black tracking-tighter">GY <span className="text-orange-500">HOMEFIX</span></span>
              </div>
              <p className="text-slate-500 text-sm leading-relaxed mb-8">
                Leading provider of home maintenance and repair services in Thane. Quality you can trust, prices you can afford.
              </p>
              <div className="flex gap-4">
                <div className="w-10 h-10 bg-white/5 rounded-xl flex items-center justify-center hover:bg-blue-600 transition-colors cursor-pointer"><Instagram size={18}/></div>
                <div className="w-10 h-10 bg-white/5 rounded-xl flex items-center justify-center hover:bg-green-600 transition-colors cursor-pointer"><MessageCircle size={18}/></div>
              </div>
            </div>
            <div>
              <h4 className="text-xs font-black uppercase tracking-[0.3em] mb-8 text-slate-500">Navigation</h4>
              <ul className="space-y-4 text-slate-400 font-bold text-sm">
                <li><button onClick={() => setView('home')} className="hover:text-blue-500 transition-colors">Home</button></li>
                <li><button onClick={() => setView('services')} className="hover:text-blue-500 transition-colors">Service List</button></li>
                <li><button onClick={() => setView('about')} className="hover:text-blue-500 transition-colors">Our Story</button></li>
                <li><button onClick={() => setView('contact')} className="hover:text-blue-500 transition-colors">Contact Support</button></li>
              </ul>
            </div>
            <div>
              <h4 className="text-xs font-black uppercase tracking-[0.3em] mb-8 text-slate-500">Core Services</h4>
              <ul className="space-y-4 text-slate-400 font-bold text-sm">
                <li>AC Repair & Refill</li>
                <li>Leakage Solutions</li>
                <li>Electrical Rewiring</li>
                <li>UPVC Window Tracks</li>
                <li>Custom Carpenter Work</li>
              </ul>
            </div>
            <div>
              <h4 className="text-xs font-black uppercase tracking-[0.3em] mb-8 text-slate-500">Office Info</h4>
              <ul className="space-y-5 text-slate-400 font-bold text-sm">
                <li className="flex items-center gap-3"><Phone size={16} className="text-blue-600"/> +91 9984281797</li>
                <li className="flex items-center gap-3"><Mail size={16} className="text-blue-600"/> info@gyhomefixservice.in</li>
                <li className="flex items-start gap-3"><MapPin size={16} className="text-blue-600 shrink-0"/> Bramhand Azad Nagar, Thane, Maharashtra</li>
              </ul>
            </div>
          </div>
          <div className="pt-12 border-t border-white/5 flex flex-col md:flex-row justify-between items-center gap-4 text-slate-600 font-bold text-[10px] uppercase tracking-widest">
            <p>© {new Date().getFullYear()} GY HomeFix Service. Thane, India.</p>
            <p>Designed for Excellence • Aman kumar 8303618319</p>
          </div>
        </div>
      </footer>

      {/* Floating Action Button (WhatsApp) */}
      <a 
        href="https://wa.me/7619049767" 
        target="_blank" 
        rel="noopener noreferrer"
        className="fixed bottom-8 right-8 z-50 bg-green-500 text-white p-5 rounded-[24px] shadow-2xl shadow-green-500/30 hover:bg-green-600 transition-all hover:scale-110 active:scale-95 group"
      >
        <MessageCircle size={32} />
        <span className="absolute right-20 top-1/2 -translate-y-1/2 bg-slate-900 text-white px-4 py-2 rounded-xl text-[10px] font-black uppercase tracking-widest opacity-0 group-hover:opacity-100 transition-opacity whitespace-nowrap pointer-events-none">
          WhatsApp Us
        </span>
      </a>

      {/* Toast Notification System */}
      {notification && (
        <div className={`fixed bottom-10 left-1/2 -translate-x-1/2 z-[100] p-5 px-8 rounded-3xl shadow-2xl border flex items-center gap-4 animate-in slide-in-from-bottom duration-300 ${
          notification.type === 'success' ? 'bg-white border-green-100 text-slate-900' : 'bg-red-950 border-red-900 text-white'
        }`}>
          <div className={`${notification.type === 'success' ? 'bg-green-500' : 'bg-red-500'} p-1.5 rounded-full text-white`}>
            {notification.type === 'success' ? <CheckCircle size={20} /> : <AlertCircle size={20} />}
          </div>
          <p className="text-sm font-black tracking-tight">{notification.message}</p>
        </div>
      )}

      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@200;300;400;500;600;700;800&display=swap');
        
        body {
          font-family: 'Plus Jakarta Sans', sans-serif;
        }

        @keyframes slide-in-from-top {
          from { transform: translateY(-20px); opacity: 0; }
          to { transform: translateY(0); opacity: 1; }
        }

        .animate-in {
          animation-duration: 0.5s;
          animation-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
        }
      `}</style>
    </div>
  );
}